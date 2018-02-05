---
layout: post
title:  "IndexedDB, implementando a persistência com o pattern Data Mapper - Parte 1"
description: Os navegadores do mercado suportam o banco de dados IndexedDB especificado pela W3C. Todavia, realizar operações de persistência através de sua API é uma tarefa um tanto árdua. Neste artigo implementaremos o padrão de projeto Data Mapper para reduzir bastante a complexidade da API do IndexedDB. É necessário que o leitor tenha algum conhecimento desta API para que aproveite melhor este artigo.
date: 2018-02-05 09:00:00 -0300
categories:
permalink: /indexeddb-implementando-persistencia-com-pattern-data-mapper/
author: flavio_almeida
tags: [javascript, indexedDB, data mapper, persistence, persistência]
image: logo.png
---

Os navegadores do mercado suportam o banco de dados <a href="https://developer.mozilla.org/pt-BR/docs/IndexedDB" target="_blank">IndexedDB,</a> especificado pela W3C. Todavia, realizar operações de persistência através de sua API é uma tarefa um tanto árdua. Neste artigo implementaremos o padrão de projeto *Data Mapper* para reduzir bastante a complexidade da API do IndexedDB. É necessário que o leitor tenha algum conhecimento desta API para que aproveite melhor este artigo.

## Prólogo

A motivação para a escrita deste artigo veio de um *coding dojo* no qual o autor foi questionado por um dos participantes o motivo de não ter abordado o pattern *Data Mapper* em seu livro <a href="https://www.casadocodigo.com.br/products/livro-cangaceiro-javascript" target="_blank">*Cangaceiro JavaScript, uma aventura no sertão da programação*</a>. Como resposta foi lhe dito que nas mais de 500 páginas do livro inevitavelmente algum assunto teria que ficar de fora. Foi então que o participante perguntou se era possível implementá-lo durante o dojo, tarefa que foi aceita por este autor e implementada em 7 minutos. 

>*É importante frisar que a implementação foi realizada em 7 minutos, porém as discussões sobre a API a ser criada levaram aproximadamente 20 minutos. Outro ponto é que apenas as operações de persistência `save()` e `list()` foram implementadas. 

Agora que já sabemos os eventos que antecederam a escrita deste artigo, vamos ao problema a ser resolvido.

## O problema

Nossa aplicação define as classes `Person` e `Animal`:

```javascript
// app/person.js
export class Person {
    constructor(name) {
        this._name = name;
    }

    get name() {
        return this._name;
    }
}
// app/animal.js
export class Animal {
    constructor(name) {
        this._name = name;
    }

    get name() {
        return this._name
    }
}
```
Precisamos persistir instâncias dessas classes no `IndexedDB`, um banco de dados presente nos navegadores do mercado:

```javascript
// app/app.js
import { Person } from './Person.js';
import { Animal } from './Animal.js';

const person = new Person('Flávio Almeida');
const animal = new Animal('Calopsita');
// como realizar a persistência no banco IndexedDB?
```

Podemos obter uma conexão do banco, criar transações e lidar com a persistência no módulo `app/app.js`, mas queremos uma solução que facilite a vida do desenvolvedor e que também possa ser reutilizada pela aplicação. Nesse sentido, podemos aplicar o padrão de projeto **Data Mapper**.

## O padrão de projeto Data Mapper

Segundo a <a href="https://en.wikipedia.org/wiki/Data_mapper_pattern">Wikipedia</a> um
Data Mapper é uma camada de acesso a dados que executa transferência bidirecional de dados entre um armazenamento de dados persistente e uma representação de dados na memória. Quanto maior a *impedância* entre esses dois mundos, maior trabalho o Data Mapper terá que realizar.

>*Impedância é a discrepância entre os dados em memória e os mesmos dados armazenados no banco.*

Vamos criar uma API que isole a complexidade de persistência com o IndexedDB. Depois de pronta, ela funcionará dessa maneira:

```javascript
// app/app.js
import { manager } from './manager.js';
import { Person } from './Person.js';
import { Animal } from './Animal.js';

(async () => {
    // configuração mínima
    await manager
        .setDbName('cangaceiro')
        .setDbVersion(2) 
        .register(
            { 
                clazz: Person,
                converter: data => new Person(data._name)
            },
            { 
                clazz: Animal,
                converter: data => new Animal(data._name)
            }
        );

    // criando instâncias
    const person = new Person('Flávio Almeida');
    const animal = new Animal('Calopsita');

    // persistindo dados
    await manager.save(person);
    await manager.save(animal);

    // buscando dados persistidos
    const persons = await manager.list(Person);
    persons.forEach(console.log);
    const animals = await manager.list(Animal);
    animals.forEach(console.log);

})().catch(console.log);
```
Sobre os métodos do objeto `manager`, nosso *Data Mapper*, temos:

* **setDbName**: define o nome do banco de dados. Quando omitido, será adotado o nome "default". Seu retorno será uma referência para a própria instância de `manager`.

* **setDbVersion**: define a versão o banco. Quando informado uma versão superior a vigente, todas as `stores` serão destruídas e criadas novamente. Quando não informado, o valor `1` será o padrão utilizado. Seu retorno será uma referência para a própria instância de `manager`.

* **register**: recebe uma lista de objetos de mapeamento com as propriedades `clazz` e `converter`. A primeira recebe a classe que terá uma `store` criada no banco. A segunda é a lógica de mapeamento dos dados retornados do banco para sua respectiva classe que será utilizado pelo método `list()`. As `stores` serão criadas apenas quando o banco for criado pela primeira vez ou quando a versão informada por `setDbVersion` for superior a versão vigente do banco. Seu retorno será uma Promise.

* **save**: recebe a instância do objeto que desejamos persistir. Internamente identifica a classe a qual o objeto pertence para decidir em qual `store` deve ser salvo. Retorna uma Promise. 

* **list**: recebe como primeiro parâmetro a classe que representa a `store` que desejarmos obter todos os seus dados armazenados.

Um ponto a se destacar é que **podemos usar a mesma conexão durante toda a aplicação sem que seja necessário fechá-la**.

Agora que já sabemos até onde queremos chegar, vamos dar início a nossa implementação.

## Implementando nosso manager

Vamos criar o módulo `app/manager.js` que terá em seu escopo as variáveis `dbName` e `dbVersion`, ambas inicializadas com valores padrões. Em seguida, definiremos a classe `Manager` com métodos acessadores para essas variáveis encapsuladas pelo módulo. No lugar de exportarmos a classe `Manager`, vamos exportar uma instância dessa classe:

```javascript
// app/manager.js
// variáveis encapsuladas pelo módulo

let dbName = 'default';
let dbVersion = 1;

class Manager {

    setDbName(name) {
        dbName = name;
        // retorna a própria instância
        return this; 
    }

    setDbVersion(version) {
        dbVersion = version;
        // retorna a própria instância
        return this; 
    }
}
// exporta a instância da classe
export const manager = new Manager();
```
>*Declarar uma variável no escopo do módulo sem exportá-la com a instrução `export` torna sua visibilidade privada, desta forma, apenas a instância de `manager` exportada pelo módulo terá acesso à variável, como é o caso das variáveis `setDbBane` e `setDbVersion`.*

Um ponto a destacar é o `return this` dos métodos da classe que criamos até agora. É esse retorno que permitirá o encadeamento das chamadas desses métodos.

Vamos criar a página `index.html` que importará o módulo `app/app.js` utilizando o sistema de importação de módulos nativo do Chrome:

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Manager</title>
</head>
<body>
<!-- importou o módulo -->
<script type="module" src="app/app.js"></script>
</body>
</html>
```

>*É necessário servir a página através de um servidor Web de sua escolha, caso contrário o carregamento dos módulos não funcionará.*

Vejamos como fica o uso inicial do nosso manager no módulo `app/app.js`:

```javascript
// app/app.js
import { manager } from './manager.js';
// encadeando as chamadas de métodos
manager
.setDbName('cangaceiro')
.setDbVersion(2)
```
>*Todos os módulos que importarem nosso `manager` trabalharão com a mesma instância. Isso é importante, pois precisamos ter centralizado em um único local sua configuração. Nesse sentido, podemos dizer que `manager` implementa o padrão de projeto `Singleton`, porém, utilizando os próprios recursos da linguagem sem grandes mistérios como no <a href="https://pt.wikipedia.org/wiki/Singleton" target="_blank">pattern original</a>.* 

Agora vamos partir para a implementação de um dos métodos mais importante da nossa API de persistência, o método `register`. 

## Implementando o método register

Vamos implementar agora a função `register`, uma dos mais importantes métodos do nosso `manager`:

```javascript
// app/manager.js
let dbName = 'default';
let dbVersion = 1;
// novo dado!
const stores = new Map(); 

class Manager {

    setDbName(name) {
        dbName = name;
        return this;
    }

    setDbVersion(version) {
        dbVersion = version;
        return this;
    }
    // novo método
    register(...mappers) {
        mappers.forEach(mapper => 
            stores.set(
                mapper.clazz.name, 
                mapper.converter
            )
        );
    }
}

export const manager = new Manager();
```
>*É necessário que o leitor tenha algum conhecimento da API do IndexedDB, por mais ínfimo que seja para que compreenda com mais clareza as vantagens da abordagem aqui utilizada.*

A função `register`, através do *Rest Operator*, recebe uma quantidade indefinida de objetos que chamaremos de `mappers` (mapeadores). Cada objeto identifica a classe e a lógica de conversão a ser aplicada toda vez que objetos dessa classe forem obtidos de uma `store`.

>*Objetos recuperados da store só possuem propriedades e nenhum método, por isso é importante definir a lógica de conversão os dados trazidos do banco para sua respectiva classe.*

Para cada item da lista de `mappers` recebido utilizaremos o valor da propriedade `clazz` como *key* e o valor da propriedade `converter` como seu valor no `Map` batizado de `stores`. Esse dado é importante, pois é através dele que saberemos quais stores serão criadas e qual lógica de conversão será utilizada ao obter seus dados.

Nosso `app/app.js` por enquanto estará assim:

```javascript
// app/app.js
import { manager } from './manager.js';
import { Person } from './Person.js';
import { Animal } from './Animal.js';

manager
.setDbName('cangaceiro')
.setDbVersion(2) 
.register(
    { 
        clazz: Person,
        converter: data => new Person(data._name)
    },
    { 
        clazz: Animal,
        converter: data => new Animal(data._name)
    }
);
```

Excelente, mas no ato de realizarmos o registro precisamos criar a conexão como banco. 

## Obtendo uma conexão 

Vamos criar uma função responsável pela criação da conexão. Ela viverá no escopo do módulo `manager.js` e apenas a instância de `Manager` poderá acessá-la. 

Como a obtenção de uma conexão é uma operação assíncrona, retornaremos uma `Promise` como resposta, reduzindo assim a complexidade de termos que trabalhar com *callbacks*. 

Vamos criar a variável `conn`, também encapsulada pelo módulo `manager.js`. Ela guardará uma referência para a conexão criada. Seu valor será atribuído através da função `createConnection`:

```javascript
let dbName = 'default';
let dbVersion = 1;
const stores = new Map(); 
// nova variável
let conn = null;

const createConnection = () => 
    new Promise((resolve, reject) => {
    
    });

// definição da classe omitido
```
Vamos solicitar à função `indexedDB.open()` a abertura da conexão com o banco usando as configurações definidas em `dbName` e `dbVersion`. Porém, não temos a garantia de que a conexão foi efetuada, motivo pelo qual precisamos lidar com os eventos `onupgradeneeded`, `onsuccess` e `onerror`:

```javascript
let dbName = 'default';
let dbVersion = 1;
const stores = new Map(); 
let conn = null;

const createConnection = () => 
    new Promise((resolve, reject) => {
        // requisitamos a abertura, um evento assíncrono!
        const request = indexedDB.open(dbName, dbVersion);

        request.onupgradeneeded = e => {};

        request.onsuccess = e => {};

        request.onerror = e => {};
    });

// código posterior omitido
```

O evento `onupgradeneeded` disponibiliza uma conexão transacional que nos permite criar todas as `stores` do banco. Utilizaremos as chaves do nosso `Map` `stores` como nome das `stores`:


```javascript
let dbName = 'default';
let dbVersion = 1;
const stores = new Map(); 
let conn = null;

const createConnection = () => 
    new Promise((resolve, reject) => {

        const request = indexedDB.open(dbName, dbVersion);

        request.onupgradeneeded = e => {

            const transactionalConn = e.target.result;
            // utilizando for...of e destructuring ao mesmo tempo
            // para ter acesso ao valor da key
            for (let [key, value] of stores) {
                const store = key;
                // se já existe, apagamos
                if(transactionalConn.objectStoreNames.contains(store)) 
                    transactionalConn.deleteObjectStore(store);
                transactionalConn.createObjectStore(store, { autoIncrement: true });
            }     
        };

        request.onsuccess = e => {
            conn = e.target.result; // guarda uma referência para a conexão
            resolve(); // tudo certo, resolve a Promise!
        }
        // lida com erros, retornando uma mensagem de alto nível
        request.onerror = e => {
            console.log(e.target.error);
            return Promise.reject('Não foi possível obter a conexão com o banco');
        }; 
    });
// código posterior omitido
```
>*A estratégia utilizada durante um eventual upgrade do banco é derrubar todas as stores para em seguida criá-las novamente.*

Excelente, agora precisamos chamar a função `createConnection` através do método `register` de `Manager`. Vamos tornar o método `async` para que possamos utilizar a instrução `await` com a Promise retornada por `createConnection`:

```javascript
// app/manager.js
// código anterior omitido

class Manager {

    setDbName(name) {
        dbName = name;
        return this;
    }

    setDbVersion(version) {
        dbVersion = version;
        return this;
    }
    // tornou o método async
    async register(...mappers) {
        mappers.forEach(mapper => 
            stores.set(mapper.clazz.name, mapper.converter));
        // utilizou a instrução await, só é possível porque
        // register é um método async
        await createConnection();
    }
}

export const manager = new Manager();
```

Agora, em `app/app.js`, criaremos um wrapper `async` para que possamos utilizar a instrução `await` com o método `register`:

```javascript
// app/app.js
import { manager } from './manager.js';
import { Person } from './Person.js';
import { Animal } from './Animal.js';
// wrapper async
(async () => {
    // instrução await!
    await manager
        .setDbName('cangaceiro')
        .setDbVersion(2) 
        .register(
            { 
                clazz: Person,
                converter: data => new Person(data._name)
            },
            { 
                clazz: Animal,
                converter: data => new Animal(data._name)
            }
        );

})().catch(console.log); // captura possíveis erros
```

>*Toda função async retorna uma Promise, independente se retornamos uma ou não. É por isso que precisamos usar `await` na chamada de `manager.register`. Além disso, lidamos com qualquer exceção através `().catch(console.log);`. Outra alternativa era usarmos a instrução `try/catch` no bloco do código, porém o autor preferiu a primeira abordagem por ser menos verbosa, além do tempo do dojo ter sido curto!*

Recarregando a página `index.html` no Chrome, na aba *Application -> Storage -> IndexedDB* podemos constatar a criação do banco.

>*Abra outra aba no Chrome para que possa verificar os dados mais atualizados, pois este autor constatou que o Chrome não realiza o refresh automático da aba Application -> Storage -> IndexedDB.*

<div style="text-align: center; margin-bottom: 20px">
<img src="data:image/png;base64,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" alt="stores criadas no banco cangaceiro" />
</div>
Agora vamos implementar o método `save`.

## Implementando o método save

O método `save` da nossa classe `Manager` também retornará uma `Promise`, pois a operação de persistência também é uma operação assíncrona:

```javascript
// app/manager.js
// código anterior omitido 
class Manager {
    // métodos anteriores omitidos
    save(object) {
        
        return new Promise((resolve, reject) => {

            // falhando rapidamente, "fail fast"
            if(!conn) return reject('Você precisa registrar o banco antes de utilizá-lo');
            
            // obtem o nome da store através do nome da classe
            const store = object.constructor.name;
            
            const request = conn
                .transaction([store],"readwrite")
                .objectStore(store)
                .add(object);
            
            // resolve a Promise no sucesso
            request.onsuccess = () => resolve();

            request.onerror = e => {
                console.log(e.target.error);
                return reject('Não foi possível persistir o objeto');
            };
        });
    }
} 
```

Vamos analisar o código anterior. A Promise retornada verifica se a variável `conn` possui algum valor e caso não possua, rejeitamos a `Promise` imediatamente. Esse *fail fast* é importante, pois sinalizará para o desenvolvedor que ele deve realizar antes o registro do banco. 

O restante do código é padrão da API do IndexedDB, todavia vale lembrar que precisamos solicitar uma transação de escrita para a `store` que desejamos armazenar nosso objeto. Aliás, obtemos o nome da `store` do objeto a ser persistido através de `object.constructor.name`:

Agora chegou a hora de implementarmos o método `list()`, aquele que retornará os dados persistidos.

## Implementando o método list 

Assim como o método `save()`, o método `list()` retornará uma `Promise`. Ele receberá como parâmetro uma classe e, a partir dela, extrairá o nome da sua respectiva `store`. Sabemos que o nome da `store` é a *key* do `Map` `stores`. É através dessa `key` que temos acesso a lógica de conversão, aquela que sabe converter os dados retornados da `store` para uma instância da sua respectiva classe. 

```javascript
// app/manager.js
// código anterior omitido 
class Manager {
    // métodos anterior omitidos
    list(clazz) {
        
        return new Promise((resolve, reject) => {
            // Identifica a store
            const store = clazz.name;
            // Cria uma transação de escrita
            const transaction = conn
                .transaction([store],'readwrite')
                .objectStore(store); 
            
            const cursor = transaction.openCursor();
            // Converter da store
            const converter = stores.get(store);
            // Array que receberá os dados convertidos
            // com auxílio do nosso converter
            const list = [];
            // Será chamado uma vez para cada 
            // objeto armazenado no banco
            cursor.onsuccess = e => {
    
                const current = e.target.result;
                // Se for null, não há mais dados
                 if(current) {
                     list.push(converter(current.value));
                     // vai para o próximo registro
                     current.continue();
                } else resolve(list);
            };
    
            cursor.onerror = e => {
                console.log(target.error);
                reject(`Não foi possível lista os dados da store ${store}.`);
            };  
        });    
    }
}
```

Excelente, nosso módulo `app/app.js` ficará assim:

```javascript
import { manager } from './manager.js';
import { Person } from './Person.js';
import { Animal } from './Animal.js';

(async () => {
    await manager
        .setDbName('cangaceiro')
        .setDbVersion(2) 
        .register(
            { 
                clazz: Person,
                converter: data => new Person(data._name)
            },
            { 
                clazz: Animal,
                converter: data => new Animal(data._name)
            }
        );
        
    const person = new Person('Flávio Almeida');
    const animal = new Animal('Calopsita');

    await manager.save(person);
    await manager.save(animal);

    const persons = await manager.list(Person);
    persons.forEach(console.log);

    const animals = await manager.list(Animal);
    animals.forEach(console.log);

})().catch(console.log);
```

Melhor organizado do que se tivéssemos utilizado diretamente a API do IndexedDB.

## E o método update? Delete?

Durante o dojo eu não implementei atualização e deleção de dados. Aliás, assunto interessante para o próximo post!

## Conclusão

O padrão de projeto Data Mapper pode ser aplicado não apenas com o IndexedDB, mas qualquer meio de persistência, inclusive banco de dados utilizados por API. 

E você? Já utilizou alguma biblioteca que faz uso desse padrão? Como lidava com persistência no IndexedDB? Deixe sua opinião.