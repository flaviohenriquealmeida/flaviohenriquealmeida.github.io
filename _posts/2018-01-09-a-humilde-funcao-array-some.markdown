---
layout: post
title:  "A humilde função Array.some"
description: Array em JavaScript é uma estrutura de dados poderosa. Funções como Array.map,Array.filter e Array.reduce resolvem uma grande quantidade de problemas do dia a dia. No entanto, há a função coadjuvante `Array.some`, um tanto esquecida, que pode simplificar bastante nosso código. Neste artigo aprenderemos a utilizá-la. 
date: 2018-01-01 06:00:00 -0300
categories:
permalink: /a-humilde-funcao-array-some/
author: flavio_almeida
tags: [javascript, array, some, recursion, refactor]
image: logo.png
---
Array em JavaScript é uma estrutura de dados poderosa. Funções como `Array.map`,`Array.filter` e `Array.reduce` resolvem uma grande quantidade de problemas do dia a dia. No entanto, há a função coadjuvante `Array.some`, um tanto esquecida, que pode simplificar bastante nosso código. Neste artigo aprenderemos a utilizá-la.

## O problema

Precisamos gerar uma lista de números aleatórios dentro de uma faixa específica. Vamos criar a função `randonFactory` que recebe o valor mínimo e máximo que define a faixa. Seu retorno será uma função parcial que ao ser chamada sempre retornará um número aleatório na faixa definida:

```javascript
const randonFactory = (min, max) => () => 
  Math.floor(Math.random() * (max - min + 1)) + min;
// nossa função geradora configurada
// para gerar um número de 1 a 10
const generate = randonFactory(1, 10);
// nosso array de números aleatórios
const randonNumbers = Array.from(new Array(4), generate);

console.log(randonNumbers);
```

Usamos a função <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Array/from" target="_blank">Array.from</a> adicionada no ES2015 (ES6) para criar um array com quatro elementos e ao mesmo tempo aplicar uma função de mapeamento para cada um dos itens que possui. Queremos mapear os quatro elementos `undefined` para um número aleatório.

Temos uma lista de números aleatórios, no entanto, podem existir elementos duplicados, algo que não faz sentido no contexto da nossa aplicação.

## Primeira solução

Uma solução é verificarmos se o novo elemento existe no array antes de adicioná-lo. Para ficar mais divertido, vamos criar uma função recursiva chamada `addNewItem`:

```javascript
const randonFactory = (min, max) => () => 
  Math.floor(Math.random() * (max - min + 1)) + min;

const generate = randonFactory(1, 10);
// recebe a lista na qual os elementos serão adicionados
// a quantidade de números a ver adicionado
const addNewItem = (randonNumbers, quantity) => {
  // quebra a chamada recursiva
  if(quantity == 0) return;
  const randon = generate();
  // variável de controle. Assume que não existe inicialmente
  let exists = false;
  // itera através de "for of", introduzido no ES2015
  for(let number of randonNumbers) {
    // se achou, muda o estsdo de exists e sai imediamente do loop
    if(number === randon) {
      exists = true;
      break;
    }
  }
  // se não existir, adiciona na lista e decrementa quantity
  if(!exists) {
    randonNumbers.push(randon);
    --quantity
  }
  // chama novamente addNewItem recursivamente
  addNewItem(randonNumbers, quantity);
}

const randonNumbers = [];
// adicionar 4 números aleatórios
addNewItem(randonNumbers, 4);
console.log(randonNumbers);
```
Se olharmos a saída de no console, teremos resultados como:

```bash
[10, 7, 6, 3]
[9, 2, 6, 7]
[6, 4, 2, 3]
```
A maior parte do código escrito foi feita para verificar a existência de um item dentro da lista. Nesse sentido, podemos usar `Array.some` para enxugar nosso código.

## Utilizando o humilde Array.some

Primeiro, camos refatormar nosso código para utilizar `Array.some` para então explicar como a função funciona:

```javascript
const randonFactory = (min, max) => () => 
  Math.floor(Math.random() * (max - min + 1)) + min;
  
const generate = randonFactory(1, 10);

const addNewItem = (randonNumbers, quantity) => {
  if(quantity == 0) return;
  const randon = generate();
  // novidade aqui 
  const exists = randonNumbers.some(item => item == randon);
  if(!exists) {
    randonNumbers.push(randon);
    --quantity
  }
  addNewItem(randonNumbers, quantity);
}

const randonNumbers = []
addNewItem(randonNumbers, 4);
console.log(randonNumbers);
```
>*Refatorar é alterar a estrutura do código sem mudar seu comportamento final.*

Antes, nosso código estava assim:

```javascript
// código anterior omitido
  let exists = false;
  // itera através de "for of", introduzido no ES2015
  for(let number of randonNumbers) {
    // se achou, muda o estsdo de exists e sai imediamente do loop
    if(number === randon) {
      exists = true;
      break;
    }
  }
// código posterior omitido
```

Com `Array.some`, mudou para:

```javascript
const exists = randonNumbers.some(item => item == randon);
```

A função `Array.some` itera em cada elemento do array aplicando uma lógica de comparação. Ela abortará a iteração imediamente assim que encontrar o primeiro item que retorne `true` na comparação. Seu retorno será `true` caso exista algum (some) elemento que se coadune com o critério de comparação. Se não nenhum elemento bater com o critério, a função retornará `false`.

## Conclusão 

A função `Array.some` é uma mão na roda toda vez que precisamos verificar se um elemento já existe em um array. Com ela, não precisamos nos preocupar com quebra do laço nem variáveis de controle.

E você? Já usava a função Array.some antes? Que tipo de problema resolveu? Deixe sua opinião.