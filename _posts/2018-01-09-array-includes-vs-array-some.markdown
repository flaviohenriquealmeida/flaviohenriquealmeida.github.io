---
layout: post
title:  "Array.includes vs Array.some"
description: Neste artigo veremos as funções Array.includes e Array.some, inclusive pegadinhas e quando aplicar uma ou outra.
date: 2018-01-08 06:00:00 -0300
categories:
permalink: /array-includes-vs-array-some/
author: flavio_almeida
tags: [javascript, array, some, includes, Array.some, Array.includes, refactor]
image: logo.png
---
Array em JavaScript é uma estrutura de dados poderosa. Funções como `Array.map`,`Array.filter` e `Array.reduce` resolvem uma grande quantidade de problemas do dia a dia. No entanto, há duas funções parecidas, mas essencialmente diferentes que fazem parte do seu arsenal. Neste artigo veremos as funções `Array.includes` e `Array.some`, inclusive pegadinhas e quando aplicar uma ou outra.

## O problema

Temos um simples array de números. Precisamos verificar se o número `15` já existe na lista:

```javascript
const numbers = [10, 11, 15, 20];
const searchNumber = 15;
let found = false;
for(number of numbers) {
  if(number === 15) {
    found = true;
    break;
  }
}
if(found) console.log(`${searchNumber} already exists!`);
```

Nada excepcional aqui, a não ser o uso de <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Statements/for...of" target="_blank">for...of</a> introduzido no ES2015 (ES6) que pode ser substituído por um loop `for` ou `while`. No entanto, podemos simplificar bastante o código utilizando um recurso introduzido no ES2017 (ES7).

## A função Array.includes

No ES2017 (ES7) foi adicionada a função `Array.includes`. Vejamos o código anterior refatorado para utilizar a nova função:

```javascript
const numbers = [10, 11, 15, 20];
const searchNumber = 15;

if(numbers.includes(searchNumber))
  console.log(`${searchNumber} already exists!`);
```
>*Refatorar é alterar a estrutura do código sem mudar seu comportamento final.*

A função `Array.includes` retornará `true` apenas se o elemento existir. Muito mais enxuto, não? Todavia, essa simplicidade pode levar o desenvolvedor ao erro.

## Resultado não esperado

Vamos repetir o exemplo da busca com `Array.includes`, mas desta vez pesquisando em uma lista de **objetos** que representam livros:

```javascript
const books = [
  { 
    name: 'Cangaceiro JavaScript',
    author: 'Flávio Almeida',
    isbn: '9788594188014'
  },
  {
    name: 'MEAN', 
    author: 'Flávio Almeida', 
    isbn: '9788555190476'
  }
];

const searchBook = { 
  name: 'Cangaceiro JavaScript',
  author: 'Flávio Almeida',
  isbn: '9788594188014'
};

if(books.includes(searchBook))
  console.log(`${JSON.stringify(searchBook)} already exists!`);
```

Por mais que a função `Array.includes` tenha funcionado no exemplo anterior, ela parece ser incapaz de encontrar nosso livro. Qual a razão desse comportamento errático?

## Comparações entre tipos

Internamente, `Array.includes` realiza uma comparação através de `===`. No entanto, comparações com `===` ou  `==` só funcionam como esperado (comparar o valor) com os tipos:

* Boolean
* Null
* Undefined
* Number
* String
* Symbol (ES2015)

Quando comparamos variáveis que referenciam o tipo `object`, o interpretador testará se elas apontam para um mesmo objeto em memória. Vejamos essa máxima em um exemplo de escopo menor:

```javascript
const object1 = { name: 'Flávio'};
const object2 = { name: 'Flávio'};
// false
console.log(object1 == object2);
// false
console.log(object1 === object2);
```

Porém, o resultado será verdadeiro se fizermos:

```javascript
const object1 = { name: 'Flávio'};
// a variável de referência object2
// passou a apontar para o mesmo objeto 
// referenciado por object1
const object2 = object1;
// true
console.log(object1 == object2);
// true
console.log(object1 === object2);
```

Nesse contexto, podemos fazer `Array.includes` retornar `true` da seguinte maneira:

```javascript
const books = [
  { 
    name: 'Cangaceiro JavaScript',
    author: 'Flávio Almeida',
    isbn: '9788594188014'
  },
  {
    name: 'MEAN', 
    author: 'Flávio Almeida', 
    isbn: '9788555190476'
  }
];

// A variável searchBook agora 
// aponta para o mesmo objeto em books[0]
const searchBook = books[0];

if(books.includes(searchBook))
  console.log(`${JSON.stringify(searchBook)} already exists!`);
```

Por mais que essa solução funcione, não faz sentido utilizá-la quando buscamos por um objeto que não sabemos sua posição ou muito menos se existe. Nessa situação, podemos recorrer ao bom e velho `Array.some`, veterano do ES5.

## Utilizando o humilde Array.some

Vejamos o código anterior utilizando a função `Array.some`:

```javascript
const books = [
  { 
    name: 'Cangaceiro JavaScript',
    author: 'Flávio Almeida',
    isbn: '9788594188014'
  },
  {
    name: 'MEAN', 
    author: 'Flávio Almeida', 
    isbn: '9788555190476'
  }
];

const searchBook = { 
    name: 'Cangaceiro JavaScript',
    author: 'Flávio Almeida',
    isbn: '9788594188014'
};

if(books.some(book => book.isbn == searchBook.isbn))
  console.log(`${JSON.stringify(searchBook)} already exists!`);
```

A função `Array.some` itera em cada elemento do array aplicando uma lógica de comparação. Ela abortará a iteração imediatamente assim que encontrar o primeiro item que retorne `true` na comparação. Seu retorno será `true` caso exista algum (some) elemento que se coadune com o critério utilizado.

O critério utilizado foi comparar a propriedade `isbn` dos objetos. Como a propriedade armazena uma `string`, comparações com `==` ou `===` utilizam o valor primitivo e não a referência. 

Ainda podemos comparar todas as propriedades do objeto com o seguinte truque:

```javascript
// código anterior omitido 
const searchBook = { 
    name: 'Cangaceiro JavaScript',
    author: 'Flávio Almeida',
    isbn: '9788594188014'
};

const bookAsJSON = JSON.stringify(searchBook);

if(books.some(book => JSON.stringify(book) == bookAsJSON))
  console.log(`${bookAsJSON} already exists!`);
```

Através de `JSON.stringify` convertemos um objeto JavaScript em um JSON que nada mais é do que a representação textual de um objeto. Sendo uma representação através de uma `string`, podemos utilizar `===` ou `==` que a comparação utilizará seu valor.

Uma das grandes vantagens de `Array.some` é podermos definir uma lógica de comparação mais rebuscada.

## Conclusão 

Nem sempre recursos mais modernos resolvem problemas do dia a dia. É importante conhecer o problema e também o recurso utilizado para resolvê-lo para não termos surpresas. 

E você? Já se surpreendeu com `Array.includes`? Já precisou utilizar `Array.some` para resolver algum problema? Deixe sua opinião.