---
layout: post
title:  "Compondo funções em JavaScript"
description: Um post sobre composição de funções e como criar uma helper function para ajudar neste processo.
date:   2017-10-09 10:00:00 -0300
categories:
permalink: /compondo-funcoes-javascript/
author: flavio_almeida
tags: [javascript, function composition, function, composição de funções]
image: logo.png
---

Não é incomum códigos como este no universo JavaScript:

```javascript
const text = ' Calopsita do Agreste ';
/* 
  Remove espaço das extremidades, coloca 
  em caixa alta e divide a string em palavras.
*/
const words = text.trim()
  .toUpperCase()
  .split(' ');

console.log(words); // ['CALOPSITA', 'DO', 'AGRESTE']
```

Não há nada de errado com essa abordagem, aliás este autor a utiliza bastante. Todavia é possível rescrevê-la com uma **estética** ainda mais funcional. 

## Uma estética mais funcional

Primeiro, vamos isolar cada operação que realizaremos dentro de uma função de mesmo nome:

```javascript
const trim = text => text.trim();
const toUpperCase = text => text.toUpperCase();
const split = separator => text => text.split(separator);
```

As funções `trim()` e `toUppercase()` recebem um texto como parâmetro retornando sua respectiva operação. Contudo, `split` deve receber um separador e retornar outra função que recebe o texto que desejamos dividir e que levará em consideração o separador:

Utilizando nossas funções ficamos com a seguinte estética:

```javascript
const trim = text => text.trim();
const toUpperCase = text => text.toUpperCase();
const split = separator => text => text.split(separator);

const words = 
  split(' ')( 
    toUpperCase(
      trim(' Calopsita do Agreste ')
    )
  );

console.log(words); // ['CALOPSITA', 'DO', 'AGRESTE']
```

Um olhar atento identificará que estamos realizando uma <a href="https://en.wikipedia.org/wiki/Function_composition" target="_blank">composição de funções</a> para atingir o resultado. 

## A função compose

Podemos criar uma função que se encarregará da composição das funções para nós. Vamos chamá-la de `compose`. Podemos realizar diferentes composições aproveitando funções já existentes:

```javascript
const trim = text => text.trim();
const toUpperCase = text => text.toUpperCase();
const split = separator => text => text.split(separator);


// já já disseco essa função!
const compose = (...fns) => value => 
  fns.reduceRight((previousValue, f) => 
      f(previousValue), value);

// a função convert é composição de outras três funções
const convert = compose(split(' '), toUpperCase, trim);
const words = convert(' Calopsita do Agreste ');
console.log(words); // ['CALOPSITA', 'DO', 'AGRESTE']
```

Vamos dissecar a função `compose` a seguir.

## Dissecando a função compose

Nossa função `compose` possui a seguinte estrutura:

```javascript
const compose = (...fns) => value => 
  fns.reduceRight((previousValue, f) => 
      f(previousValue), value);
```
Através do <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Functions/rest_parameters" target="_blank">REST Operator</a>, a função `compose` recebe quantas funções quisermos utilizar na composição. Seu retorno será uma função que recebe um único valor, aquele que será utilizado pela função composta resultante, no caso, que guardamos na variável `convert`. Mas como aplicaremos cada função ao valor recebido?

A ideia é **reduzirmos** todas as funções há apenas uma e nada mais justo do que usarmos a função <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Array/ReduceRight" target="_blank">reduceRight</a> presente em todo Array em JavaScript. Ela é idêntica ao `reduce`, mas itera do último elemento até o primeiro. Faz sentido, porque queremos que as funções sejam aplicadas da direita para a esquerda, começando por `trim()`, `toUpperCase()` e por fim `split(' ')`.

`reduceRight` recebe dois parâmetros. O primeiro é uma função que será chamada `N` vezes para cada elemento do array. Ela nos dá acesso ao valor anterior da redução e ao item que estamos iterando no momento. Já o segundo parâmetro é o valor inicial que será levado como valor anterior da redução na primeira vez que sua lógica for invocada. Quando passamos `value`, estamos deixando claro que o valor inicial será o valor passado para a função final retornada por `compose`, em nosso caso, a string `' Calopsita do Agreste '`. 

>Usamos reduceRight no lugar de reduce porque queremos estar fiéis à função matemática. Nela, o resultado da função mais interior é passado para a função mais exterior e assim por diante.

Por fim, a lógica do `reduceRight` será aplicar cada função da composição. Como usamos arrow function, o retorno de cada função será considerado como `previousValue` para a próxima função e assim por diante. 

A composição que vimos tem um "irmãzinha", uma outra forma de composição chamada `pipe`.

## Pipe de funções

Em uma composição através de `pipe`, as funções são aplicadas da esquerda para a direta, isto é, o resultado da função mais externa é passado para a mais interna e assim por diante. Vejamos um exemplo:

```javascript
const trim = text => text.trim();
const toUpperCase = text => text.toUpperCase();
const split = separator => text => text.split(separator);

const pipe = (...fns) => value => 
  fns.reduce((previousValue, f) => 
      f(previousValue), value);

// trim será aplicado, depois toUpperCase e por fim split
const convert = pipe(trim, toUpperCase, split(' '));
const words = convert(' Calopsita do Agreste ');
console.log(words); // ['CALOPSITA', 'DO', 'AGRESTE']      
```

## Conclusão

Realizar composição de funções em JavaScript é uma tarefa corriqueira, principalmente quando o programador tem forte inclinação pelo paradigma funcional.