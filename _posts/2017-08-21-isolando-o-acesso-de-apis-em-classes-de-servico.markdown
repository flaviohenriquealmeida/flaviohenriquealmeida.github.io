---
layout: post
title:  "Isolando o acesso de API's em classes de serviço"
description: Não é incomum aplicações web buscarem e enviarem dados através de uma API. Mas nem tudo é perfeito; durante esse processo a rede pode cair ou o servidor pode estar fora do ar. Dentro de tanta incerteza, nada mais justo do que informar o usuário que houve algum problema.
date:   2017-08-22 08:30:00 -0300
categories:
permalink: /isolando-acesso-api-classes-servico/
author: flavio_almeida
tags: [node, javascript, boas-praticas, service, api, oop]
image: logo.png
---

Não é incomum aplicações web buscarem e enviarem dados através de uma API. Mas nem tudo é perfeito; durante esse processo a rede pode cair ou o servidor pode estar fora do ar. Dentro de tanta incerteza, nada mais justo do que informar o usuário que houve algum problema.

Vejamos um exemplo que envia um `JSON` através do método `POST` para uma API hipotética utilizando *vanilla* JavaScript:

```javascript
// main.js

// é nóis cangaceiro!
const cangaceiro = { nome: 'Flávio', bando: 'cangaceiros javascript'};

// configurando o header da requisição
const headers = new Headers();
headers.append('Content-Type', 'application/json');

// preparando a configuração necessária para realizara a requisição
const config  = {
    method: 'post',
    headers,
    data: JSON.stringify(cangaceiro),
};

// utiliza API Fetch passando o endereço da API e a configuração
fetch('http://seu-dominio/cangaceiro', config)
    .then(() => {
      // tratamento necessário quando usamos Fetch API
      // se não estiver ok, rejeita a Promise lançando uma exceção
      if(!res.ok) throw new Error(res.statusText);
      // se nenhum exceção foi lançada, retorna a própria resposta
      return res;      
    })
    .then(() => alert('Operação realizada com sucesso')
    .catch(err => {
        console.log(err);
        alert('Não foi possível realizar a operação. Tente mais tarde.');
    });
```

No exemplo anterior, no sucesso da operação a mensagem "Operação realizada com sucesso" é exibida. No fracasso, logamos o valor de `err` enviado pelo servidor e exibimos uma mensagem de erro padrão para o usuário. Não faz sentido exibir para ele `err`, pois é uma mensagem de baixo nível que só diz respeito ao desenvolvedor.

A abordagem acima deixa a desejar. Vejamos:

* Repetiremos o endereço da API e configuração do cabeçalho em todos os lugares que acessam a API.
* Precisaremos fazer sempre `console.log()` para logar o erro vindo do servidor para o desenvolvedor e precisaremos em seguida fornecer uma mensagem de alto nível de erro para o usuário.

Podemos melhorar o código que vimos com uma ajudinha do paradigma da Orientação a Objetos. Por exemplo, <a href="http://hipsters.tech/single-page-applications-hipsters-16/" target="_blank">frameworks SPA</a> do mercado como Angular já propõem uma solução para o problema que é isolar toda a complexidade do código em classes de serviço. Por mais que estejamos escrevendo um código vanilla JavaScript nada nos impede de fazermos o mesmo. Boa prática é boa prática independente de framework ou não.

## Isolando a API em uma classe de serviço

A classe do serviço centraliza uma série de informações a respeito da API, cabeçalhos e métodos da requisição, inclusive é a responsável por logar e fornecer uma mensagem de alto nível para as operações, livrando o cliente da API de ter que lidar com esses detalhes.

Vejamos um exemplo:

```javascript
// CangaceiroService.js
class CangaceiroService {

    constructor() {
      
        this.domain = 'http://seu-dominio';
        this.headers = new Headers();
        this.headers.append('Content-Type', 'application/json');
    }

    salva(cangaceiro) {
    
        const config = { 
            headers: this.headers, 
            method: 'post', 
            data: JSON.stringify(cangaceiro) 
        };

        // retorna a Promise
        return fetch(`${this.domain}/${cangaceiro}`, config)
            .then(this._handleErrors)
            .catch(err => {
                // logou erro original
                console.log(err);
                // lançou uma exceção de alto nível que pode ser exibida para o usuário
                throw new Error('Não foi possível realizar a operação. Tente mais tarde');
            });
    }
    // pode ser usados por outros métodos
    _handleErrors() {

      if(!res.ok) throw new Error(res.statusText);
      return res;      
    }
}
```

Agora, em todo lugar que o serviço for necessário,  basta fazermos:

```javascript
// main.js

const cangaceiro = { nome: 'Flávio', bando: 'cangaceiros javascript'};

new CangaceiroService()
    .cadastra(cangaceiro)
    .then(() => alert('Operação realizada com sucesso')
    .catch(err => alert(err));
```

Muito mais fácil de ler e de manter do que a versão anterior. 

## Conclusão 

Isolar o acesso de API's em classes de serviço facilita a manutenção e legibilidade do nosso código. É uma prática que aplicada por diversos frameworks do mercado e que podemos aplicar com vanilla JavaScript, inclusive na plataforma Node.js.