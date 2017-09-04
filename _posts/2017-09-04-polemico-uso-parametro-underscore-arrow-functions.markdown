---
layout: post
title:  "O polêmico uso do parâmetro underscore em arrow functions"
description: Abordarei uma prática que permite deixar ainda mais simples a declaração de arrow functions introduzidas no ECMAScript 2015 (ES6). No entanto, essa prática é um tanto polêmica.
date:   2017-09-04 08:30:00 -0300
categories:
permalink: /polemico-uso-parametro-underscore-arrow-functions/
author: flavio_almeida
tags: [javascript, arrow function, underscore, es6, es2015]
image: logo.png
---
Abordarei uma prática que permite deixar ainda mais simples a declaração de *arrow functions* introduzidas no ECMAScript 2015 (ES6). No entanto, essa prática é um tanto polêmica.

## Declarando arrow functions sem parâmetros

Temos um exemplo de uma *function declaration* criada através de *arrow function* que não recebe parâmetro algum:

```javascript
const funcaoSemParametro = () => alert('oi');
funcaoSemParametro();
```

Somos obrigados a utilizar `()` mesmo para aquelas funções que não recebem parâmetro. Porém, podemos evitar o uso de `()` **convencionando** que arrow functions que não recebem parâmetro devem declarar o parâmetro **underscore**:

```javascript
const funcaoSemParametro = _ => alert('oi');
funcaoSemParametro();
```

Vejamos outro exemplo no qual uma função recebe outra como parâmetro:

```javascript
const chamaOutraFuncao = fn => fn();

// versão sem underscore
chamaOutraFuncao(() => alert('oi'));

// versão com underscore
chamaOutraFuncao(_ => alert('oi'));
```

Temos uma sintaxe mais tersa com esta abordagem, contudo sua aplicação é questionável.

## O underscore continua sendo um parâmetro da função

Por mais que tenhamos usado *underscore*, ele **continua sendo um parâmetro válido** para uma função que não deveria ter parâmetro algum:

```javascript
// exemplo de acesso ao parâmetro
const funcaoSemParametro = _ => {
    alert('oi');
    console.log(_); // undefined
};
funcaoSemParametro();
```

No exemplo acima, temos acesso ao parâmetro, inclusive podemos acessá-lo dentro da função, mesmo que não tenha recebido parâmetro algum.

Outro ponto é que o uso de *underscore* pode se chocar com a famosa biblioteca <a href="http://underscorejs.org/" target="_blank">underscore</a> que usa o `_` como *alias*.

## Conclusão

E você? Usaria ou não essa abordagem? Deixe seu comentário para podermos engrandecer ainda mais essa discussão.

