---
layout: post
title:  "Lidando com o módulo mysql na plataforma Node.js"
description: XXXXXX. 
date:   2017-12-18 07:00:00 -0300
categories:
permalink: /lidando-com-modulo-mysql-na-plataforma-node/
author: flavio_almeida
tags: [javascript, mysql, connection pool, dao pattern]
image: logo.png
---

Podemos realizar a comunicação da nossa aplicação Node.js com o banco MySQL através de frameworks de ORM como <a href="http://docs.sequelizejs.com/" target="_blank">Sequelize</a> ou <a href="https://github.com/balderdashy/waterline" target="_blank">Waterline</a>. Inclusive, podemos realizar o acesso através do módulo <a href="https://github.com/mysqljs/mysql" target="_blank">mysql</a>, um cliente puramente escrito em Node.js. Apesar deste último não ter um grau de abstração tão alto quanto os demais frameworks citados, ele nos fornecesse uma experiência mais nativa, porém verbosa. Neste artigo veremos algumas práticas que podem tornar o uso do módulo mysql uma experiência mais primorosa no que diz respeito ao gerenciamento de conexões e persistência.

## O problema

Temos como exemplo um trecho de <a href="https://pt.wikipedia.org/wiki/C%C3%B3digo_espaguete" target="_blank">código espaguete</a> definindo uma rota do Express que ao ser acessada consulta o banco e retorna uma lista de produtos:

```javascript
// app/api/product.js
const mysql = require('mysql');

module.exports = app => {
    /* 
        Cria um pool de conexão. Busca o host, user e password 
        de variáveis de ambiente do sistema operacional
    */ 
    const pool  = mysql.createPool({
        connectionLimit : 10,
        host : process.env.HOST,
        user : process.env.USER,
        password : process.env.PASSWORD,
        database : process.env.DATABASE
    });

    // código anterior omitido 
    app.get('/products', (req, res, next) => {

        pool.getConnection((err, connection) => {
            // se houver, passa o erro para o middleware que lida com erro 
            if(err) return next(err);
            connection.query('SELECT * FROM products', (err, products) => {
                // devolve a conexão para o pool
                connection.release();
                // se houver, passa o erro para o middleware que lida com erro 
                if(err) return next(err);
                // devolve a resposta
                res.json(products);
            });
        });
    });    
};
```

Não precisamos meditar muito para enxergarmos que o módulo `product.js` esta com muita responsabilidade. São elas:

1. Criar o pool de conexões
2. Obter uma conexão do pool
3. Preparar e executar a query 
4. Devolver a conexão para o pool após realizar a query 

Para complicar mais um pouco, além dessas responsabilidades, precisamos fechar o pool (com todas as suas conexões) elegantemente toda vez que a aplicação for finalizada. E agora?

Vamos resolver o primeiro problema da lista, o da criação do pool de conexões. 

## O módulo PoolFactory

Vamos isolar a criação e configuração do pool no módulo `pool-factory`:

```javascript 
// app/config/pool-factory.js 

const mysql = require('mysql');

const pool = mysql.createPool({
    connectionLimit,
    host : process.env.HOST,
    user : process.env.USER,
    password : process.env.PASSWORD,
    database : process.env.DATABASE
});

console.log('pool => criado');

```
Podemos até disparar um evento toda vez que uma conexão do pool for devolvida através da chamada da funcão `release` que toda conexão do pool possui:

```javascript 
// app/config/pool-factory.js 

const mysql = require('mysql');

const pool = mysql.createPool({
    connectionLimit: 10,
    host : process.env.HOST,
    user : process.env.USER,
    password : process.env.PASSWORD,
    database : process.env.DATABASE
});

console.log('pool => criado');

pool.on('release', () => console.log('pool => conexão retornada'));

```
Toda vez que a aplicação for finalizada, fecharemos o pool com todas as suas conexões. Para isso, através do objeto implícito `proccess` disponibilizado pelo Node.js, escutaremos ao evento `SIGINT` disparado quando a aplicação é finalizada:

```javascript 
// app/config/pool-factory.js 

const mysql = require('mysql');

const pool = mysql.createPool({
    connectionLimit,
    host : process.env.HOST,
    user : process.env.USER,
    password : process.env.PASSWORD,
    database : process.env.DATABASE
});

console.log('pool => criado');

pool.on('release', () => console.log('pool => conexão retornada')); 

process.on('SIGINT', () => 
    pool.end(err => {
        if(err) return console.log(err);
        console.log('pool => fechado');
        process.exit(0);
    })
); 

module.exports = pool;
```

Excelente, para criarmos nosso pool basta fazermos:

```javascript 
// exemplo!
const pool = require('./pool-factory');
```

Agora podemos atacar os itens 2 e 4:

2. Obter uma conexão do pool
4. Devolver a conexão para o pool após realizar a query 

Solucionamos ambos construindo um middleware.

## Connection Middleware

A lógica do nosso middeware será a seguinte. Independente da rota da aplicação acessada, antes que ela seja processada, nosso middleware entrará em ação criando uma conexão para em seguida adicioná-la na requisição que esta sendo processada. Para isso, ele dependerá de um pool de conexões configurado, algo que já temos.

Depois de adicionarmos a conexão na requisição, chamaremos a função `next` fazendo com que os próximos middlewares da pilha sejam processados. Isso significa que a conexão estará disponíveis para os demais middlewares através de `req.connection`:

```javascript 
// connection-middleware.js

// o módulo depenende de um pool de conexões
module.exports = pool => (req, res, next) => {

    pool.getConnection((err, connection) => {
        /* 
            Se der pau, passa o erro para o próximo middleware 
            que saiba lidar com erros. 
        */
        if(err) return next(err);
        console.log('pool => obteve conexão');
        // adicionou a conexão na requisição
        req.connection = connection;
        // passou a bola para os próximos middlewares
        next();
    });
};
```

Todavia, precisamos devolver a conexão para o pool. Mas quando faremos isso? Podemos realizar essa operação quando a aplicação tiver terminado de enviar a resposta. Conseguimos isso facilmente escutando o evento `finish` da resposta par então chamarmos a função `req.connection.release`:

```javascript 
// connection-middleware.js

module.exports = pool => (req, res, next) => {

    pool.getConnection((err, connection) => {
        if(err) return next(err);
        console.log('pool => obteve conexão');
        // adicionou a conexão na requisição
        req.connection = connection;
        // passou a bola para os próximos middlewares
        next();
        // devolve a conexão para o pool no final da resposta
        res.on('finish', () => req.connection.release());
    });
};
```
Nosso middleware precisará ser registrado **antes das configurações de rotas** da nossa aplicação:

```javascript
// app/config/express-config.js
const express = require('express')
, app = express()
, pool = require('./pool-factory')
, connectionMiddleware = require('./connection-middleware');

app.use(connectionMiddleware(pool));

// outras configurações omitidas
```

Como fica nosso `product.js`?

```javascript
// app/api/product.js

// não depende mais do módulo mysql 

app.get('/products', (req, res) => {

    // não me importa de onde vem a conexão, só preciso de uma conexão!
    req.connection.query('SELECT * FROM products', (err, products) => {

        if(err) return next(err);
        res.json(products);

        // não preciso me preocupar em devolver a conexão para o pool
    });
});
```

Veja como reduzimos a complexidade do nosso código. Simplesmente utilizamos `req.connection` sem termos de nos preocupar de onde ela veio muito menos de lembrarmos de realizar a chamada de `req.connection.release()`. Em suma, temos sempre uma nova conexão no início de qualquer requisição e no final da requisição termos a certeza de que ela será devolvida para o pool. Todavia, nosso código pode ficar ainda melhor. 

## O padrão de projeto DAO

Por mais que tenhamos isolado a complexidade de se lidar com a conexão, em todo lugar que precisarmos de uma lista de produtos teremos que reescrever o SQL e lidar com o padrão *error first-callback** que a API da conexão utilizada, aliás, padrão bem difundido em diversas API do mundo Node.js. 

Uma solução é aplicarmos o padrão de projeto DAO (Data Access Object) para isolar os detalhes de persistência de um produto. Vamos criá-lo:

```javascript
// app/api/product-dao.js

class ProductDao {

    constructor(connection) {

        this._connection = connection;
    }
}
```

Nossa classe `ProductDao` depende de uma conexão para realizar sua tarefa. Nada mais justo do que recebê-la em seu `constructor`. 

Agora, só nos resta implementar o método `list()`:


```javascript
// product-dao.js
class ProductDao {

    constructor(connection) {

        this._connection = connection;
    }

    list() {

        return new Promise((req, res) => 
            this._connection.query('SELECT * FROM products', (err, products) => {
                if(err) return reject(err);
                resolve(products);
            })
        );
    }
    // outros métodos de persistência
}
```

Nosso método `list` retorna uma Promise, isto é, quem for utilizar nosso DAO não precisará lidar com a estrutura *error-first callback*, excelente! 

Por fim, nosso `product.js` ficará assim:

```javascript
// app/api/product.js

const ProdutoDao = require('./product-dao');

app.get('/products', (req, res, next) => {

    new ProductDao(req.connection)
        .list()
        .then(products => res.json(products))
        .catch(err => next(err));
});
```

## Conclusão

Por mais que não estejamos utilizando algum framework de persistência da plataforma Node.js, nada nos impede de estruturarmos nosso código para que facilite sua legibilidade e manutenção. Aliás, lidar com o módulo `mysql` é realidade de muitos desenvolvedores que precisam ter controle fino de tudo o que ocorre em termos de persistência dentro da plataforma.