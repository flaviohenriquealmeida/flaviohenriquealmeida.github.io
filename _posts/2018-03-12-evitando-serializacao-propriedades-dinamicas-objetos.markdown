---
layout: post
title:  "Evitando a serialização de propriedades dinâmicas de objetos"
description: Existem várias maneiras de adicionarmos *flags* de controle em objetos JavaScript. Uma delas é modificando o objeto diretamente. Todavia, ao serializarmos os dados para serem enviados para uma API, a propriedade estará presente.
date: 2018-03-12 08:00:00 -0300
categories:
permalink: /evitando-serializacao-propriedades-dinamicas-objetos/
author: flavio_almeida
tags: [javascript, JSON.stringify, serialization, Object.defineProperty]
image: logo.png
---

## O problema

Existem várias maneiras de adicionarmos *flags* de controle em objetos JavaScript. Uma delas é modificando o objeto diretamente:

```javascript
// dados que podem vir de uma API web
const photo = {
    title: 'Cangaceiro JavaScript',
    author: 'Flávio Almeida'
}
// propriedade de controle que define um identificador único
photo._key = 1;
```
Todavia, ao serializarmos o dado para ser enviado para uma API, a propriedade `_key` estará presente:

```javascript
const serializedPhoto = JSON.stringify(photo);
console.log(serializedPhoto);
/* 
{
    title: 'Cangaceiro JavaScript', 
    author: 'Flávio Almeida', 
    _key: 1
}
*/
```
Isso não seria um problema caso a API seguisse a boa prática de pinçar (pluck) apenas as propriedades que espera receber. Mas caso a API não siga esse caminho, fica evidente que não podemos enviar a propriedade `_key`. 

>Serializar o JSON em disco na plataforma Node.js também seria um problema, pois a propriedade também faria parte do dado serializado. 

Uma solução é apagarmos a propriedade adicionada dinamicamente antes da serialização:

```javascript
// código anterior omitido
delete photo._key; // apagando a propriedade
const serializedPhoto = JSON.stringify(photo);
delete serializedPhoto._key;
console.log(serializedPhoto);
```

>A motivação para o post veio do problema enfrentado por um colega em sua aplicação feita com Ionic 3. Uma das bibliotecas utilizadas por ele modificava diretamente o modelo associado ao componente o que acabava lhe causando problemas na arquitetura definida por ele.

O exemplo acima não causaria grandes problemas de performance, mas o cenário seria diferente caso tivéssemos que iterar em um grande lista de objetos para então deletarmos a propriedade `_key` de cada uma deles. A boa notícia é que há outra solução.

## Definindo propriedades não iteráveis

Uma solução para o problema anterior é adicionarmos no objeto a propriedade `_key` como não enumerável. Uma propriedade não enumerável não será enxergada por repetições `for...in` ou pelo método `Object.keys()`. Como a função `JSON.stringify()` utiliza alguma estrutura de iteração para construir o JSON resultante, propriedades não enumeráveis serão ignoradas.


## Um pouco sobre Object.defineProperty

Criamos propriedades não iteráveis com auxílio da função <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty" target="_blank">`Object.defineProperty`</a>. Ela recebe como primeiro parâmetro o objeto que desejamos modificar e como segundo o nome da propriedade que será adicionada. Por fim, um objeto descritor (descriptor) é passado como último parâmetro. É nele que indicamos que a propriedade é não enumerável através de `enumerable: false`, inclusive seu valor através de `value: 1`:

Modificando nosso código:

```javascript
const photo = {
    title: 'Cangaceiro JavaScript',
    author: 'Flávio Almeida'
}

Object.defineProperty(photo, '_key', { 
    enumerable: false, 
    value: 1
});

const serializedPhoto = JSON.stringify(photo);
console.log(serializedPhoto);
/* 
{
    title: 'Cangaceiro JavaScript', 
    author: 'Flávio Almeida', 
}
*/
```
O JSON resultante não contém mais a propriedade `_key`, excelente.

## Conclusão 

Podemos adicionar dinamicamente propriedades não enumeráveis de em objetos através de `Object.defineProperty`. Uma consequência dessas propriedades é que elas não farão parte do JSON criado através da função `JSON.stringify`. 

O função `Object.defineProperty` permite definir ainda outras características da propriedade adicionada dinamicamente. E você? Já precisou utilizar essa função antes? Qual problema você tentou resolver? Deixe sua opinião.