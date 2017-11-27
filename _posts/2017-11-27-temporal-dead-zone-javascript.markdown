---
layout: post
title:  "Temporal Dead Zone no JavaScript"
description: Neste artigo veremos como o Temporal Dead Zone (TDZ) pode afetar nosso código quando declaramos variáveis com let e os cuidados que o desenvolvedor precisa ter.
date:   2017-11-27 08:00:00 -0300
categories:
permalink: /temporal-dead-zone-no-javascript/
author: flavio_almeida
tags: [javascript, tdz, temporal dead zone, let, hoisting]
image: logo.png
---

Neste artigo veremos como o *Temporal Dead Zone (TDZ)* pode afetar nosso código quando declaramos variáveis com `let` e os cuidados que o desenvolvedor precisa ter.

## Declaração com var 

Vejamos o seguinte exemplo que declara uma variável através de `var`:

```javascript
function minhaFuncao() {

    alert(nome);
    var nome = 'Almeida';
}
minhaFuncao();
```

Por mais que tenhamos escrito um código desta maneira, quando ele for processado por uma engine JavaScript será interpretado desta forma:

```javascript
// COMO O INTERPRETADOR LÊ NOSSO CÓDIGO

function minhaFuncao() {

    var nome;
    alert(nome);
    nome = 'Almeida';
}
minhaFuncao();
```

O que acabamos de ver é o *hoisting* (içamento) da variável `nome` para o início do bloco no qual foi declarada. Dessa maneira, temos a declaração `var nome;`, que não possui qualquer atribuição. Por isso o `alert(nome)` exibe *undefined* em vez de um erro alegando que a variável não existe. 

Contudo, quando usamos `let`, introduzida a partir do ES2015 (ES6) precisamos ficar atentos. Lembrando que variáveis declaradas com `let` possuem escopo de bloco e não podem ser redeclaradas dentro de um mesmo escopo.

## Declarações com let

Variáveis declaradas com `let` também são içadas (*hoisting*). Contudo, seu acesso só é permitido após receberem uma atribuição; caso contrário, teremos um erro. 

Vejamos o exemplo anterior, desta vez usando `let`:

```javascript
function minhaFuncao() {
    
    alert(nome);
    let nome = 'Flávio';
}

minhaFuncao();
```

Quando executamos nosso código, receberemos a mensagem de erro:

```
Uncaught ReferenceError: nome is not defined
```

Como foi dito, variáveis declaradas com `let` também são içadas, então nosso código estará assim quando interpretado:

```javascript
// COMO O INTERPRETADOR LÊ NOSSO CÓDIGO

function minhaFuncao() {
    
    let nome;
    alert(nome);
    nome = 'Flávio';
}

minhaFuncao();
```

Mas o que explica esse comportamento diferente com variáveis declaradas com `let`? É o que veremos a seguir.

## Temporal Dead Zone

Entre a declaração da variável com `let` e sua atribuição há o *Temporal Dead Zone (TDZ)*. Qualquer acesso feito à variável nesse espaço de tempo resultará em `ReferenceError`. Contudo, é um erro benéfico, pois acessar uma variável antes de sua atribuição é raramente intencional em uma aplicação. 

Nada nos impede de deliberadamente fazermos isso:

```javascript
function minhaFuncao() {
    let nome = undefined;
    alert(nome);
    nome = 'Flávio';
}

minhaFuncao();
```

O resultado de `alert` será `undefined`. Faz todo sentido, porque acessamos a variável `nome` após receber uma atribuição, isto é, fora do *Temporal Dead Zone*.

## Curiosidade

Variáveis declaradas com `const` não são afetadas pela `TDZ`, pois obrigatoriamente precisam ser inicializadas durante sua declaração.

## Conclusão

A introdução de `let` foi uma adição louvável no mundo JavaScript, todavia precisamos estar atentos ao *temporal dead zone*, área de veto ao acesso de variáveis que não existia antes do ES2015 (ES6).