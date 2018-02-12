---
layout: post
title:  "Suportando Method Chaining em uma API já existente"
description: Neste artigo aprendemos a suportar *Method Chaining* em uma API sem suporte a esse recurso.
date: 2018-02-12 08:00:00 -0300
categories:
permalink: /suportando-method-chaining-em-api-ja-existente/
author: flavio_almeida
tags: [javascript, pattern, method chaining, fluent interface]
image: logo.png
---

Neste artigo aprendemos a suportar *Method Chaining* em uma API sem suporte a esse recurso.

## O Problema 

O IndexedDB disponibiliza uma API para que possamos persistir objetos. Vejamos um trecho desta API em ação:

```javascript
// conexão criada com o banco antes
const store = 'books';
object = { name: 'Cangaceiro JavaScript', isbn: 9788594188014 };

const request = connection
    .transaction([store],"readwrite")
    .objectStore(store)
    .add(object);
        
request.onsuccess = () => console.log('persistiu com sucesso');

request.onerror = e => console.log(e.target.error);
```

Queremos encadear as chamadas de `onsuccess` e `onerror`. Em outras palavras, queremos o suporte à **Method Chaining**. Mas para que isso seja possível, precisamos criar uma requisição encadeável. Por padrão, o IndexedDB não suporta requisições que suportem *chaining*, no entanto, podemos criar um wrapper que encapsule a requisição e permita utilizar o recurso desejado:

```javascript
// conexão criada com o banco antes
const store = 'books';
object = { name: 'Cangaceiro JavaScript', isbn: 9788594188014 };

// cria um wrapper a partir da requisição
ChainableRequest.of(connection
    .transaction([store],"readwrite")
    .objectStore(store)
    .add(object)
).success(() => console.log('persistiu com sucesso'))
).error(e => console.log(e.target.error))
```

No código anterior temos a chamada de `ChainableRequest.of()`, mas a classe `ChainableRequest` não existe. Precisamos implementá-la.

## Implementando a classe ChainableRequest. 

A classe `ChainableRequest` receberá em seu `constructor` o objeto request que encapsulará:

```javascript
class ChainableRequest {

    constructor(request) {
        this._request = request;
    }
}
```
>*A classe ChainableRequest é um wrapper que encapsula o objeto original.*

Excelente, mas do jeito que construímos nossa classe, precisaremos utilizar o operador `new` para criar instâncias de `ChainableRequest`. Podemos evitar o uso do operador através do método estático `of`:

```javascript
class ChainableRequest {

    constructor(request) {
        this._request = request;
    }

    static of(request) {
        return new ChainableRequest(request);
    }
}
```

Da maneira que organizamos o código é o próprio método `of()` que se encarregará de criar a instância de `ChainableRequest` lidando com o operador `new`. Vamos continuar nossa implementação. 

## Criando métodos que retornam a própria instância

Vamos criar um método para cada propriedade de uma requisição do IndexedDB:


```javascript
class ChainableRequest {

    constructor(request) {
        this._request = request;
    }

    static of(request) {
        return new ChainableRequest(request);
    },

    success(fn) {
        this._request.onsuccess = fn;
        return this;
    }

    error(fn) {
        this._request.onerror = fn;
        return this;
    }
}
```

Vejam que os métodos `success` e `error` recebem a lógica que será aplicada nas respectivas propriedades `_request.onsuccess` e `_request.onerror`. Todavia, o retorno de ambos é a própria instância de `ChainableRequest`. É esse retorno que permitirá as chamadas chamadas encadeadas. Com a classe completa, o código proposto no início deste artigo funcionará:

```javascript
// conexão criada com o banco antes
const store = 'books';
object = { name: 'Cangaceiro JavaScript', isbn: 9788594188014 };
ChainableRequest.of(connection
    .transaction([store],"readwrite")
    .objectStore(store)
    .add(object)
).success(() => console.log('persistiu com sucesso'))
).error(e => console.log(e.target.error))
```

O ganho na legibilidade pode ser ainda maior dependendo da quantidade de métodos encadeáveis utilizados para resolver o problema em questão.

## Conclusão

Não é pelo fato de uma API não ser encadeável que ela nos privará desse recurso. Com um pouco de código podemos criar uma pequena camada sob a API já existente possibilitando chamadas encadeáveis. 

E você? Já aplicou essa estratégia antes? Tem algum exemplo de outra API no qual essa estratégia se torna ainda mais interessante. Deixe seu comentário.