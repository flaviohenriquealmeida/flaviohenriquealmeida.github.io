---
layout: post
title:  "De vanilla JavaScript para RxJS"
description: 
date:   2017-11-27 17:00:00 -0300
categories:
permalink: /de-vanilla-javascript-para-rxjs/
author: flavio_almeida
tags: [javascript, rxjs, functional, reactive]
image: logo.png
---

Essa semana um dos estagiários aqui da <a href="https://www.caelum.com.br/" target="_blank">Caelum</a> me perguntou se valia a pena investir no estudo do <a href="https://github.com/Reactive-Extensions/RxJS" target="_blank">Rxjs</a>. Minha resposta poderia ser simplesmente um "sim" ou um "não". Todavia, não poderia dar uma resposta tão direta assim, pois ela não teria efeito transformador em seu conhecimento. Foi então que decidi expor uma problema para solucionarmos com vanilla JavaScript para no final, demonstrar o mesmo código utilizando Rxjs. Ele curtiu a ideia. 

## Um problema

Temos uma simples página que exibe um botão apenas:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width">
        <title>App</title>
    </head>
    <body>
        <button id="btn">Import</button>
        <script type="module" src="js/app.js"></script>
    </body>
</html>
```

>*O código anterior utiliza importação nativa de módulos do ES2015. Você pode consultar o artigo  <a href="http://cangaceirojavascript.com.br/importacao-nativa-modulos-browser/" target="_blank">"Importação nativa de módulos no browser"</a> deste mesmo autor para saber mais sobre este assunto.*

Quando o usuário clicar no botão, precisamos buscar de uma API as negociações da semana atual e da semana anterior exibindo o resultado no console. Porém, se o usuário clicar mais de uma vez no botão dentro de uma janela de tempo de meio segundo, independente da quantidade de cliques realizados, apenas um deverá ser processado. Por fim, ele só poderá realizar a busca três vezes e qualquer tentativa de realizar uma nova busca deve ser ignorada. 

## Solucionando com vanilla JavaScript

Primeiro, vamos criar o arquivo `js/api.js`:

```javascript
// js/api.js

// função utilitária para lidar com o status da requisição
const handleError = res => {
  if(!res.ok) throw Error(res.statusText);
  return res;
};

// busca as negociações da semana 
const getNegotiationsFromWeek  = () =>
    fetch('http://localhost:3000/negociacoes/semana')
    .then(res => handleError(res))
    .then(res => res.json())
    .catch(err => {
        console.log(err);   
        return Promise.reject('getNegotiationsFromWeek: failure!');
    });

// busca as negociações da semana anterior
const getNegotiationsFromPreviousWeek = () =>
    fetch('http://localhost:3000/negociacoes/anterior')
    .then(res => handleError(res))
    .then(res => res.json())
    .catch(err => {
        console.log(err);
        return Promise.reject('getNegotiationsFromPreviousWeek: failure!');
    });

/* 
    Combina em um array de única dimensão as 
    negociações da semana atual a anterior
*/
export const getNegotiations = () => 
    Promise
    .all([getNegotiationsFromWeek(), getNegotiationsFromPreviousWeek()])
    .then(data => 
        data.reduce((negotiations, array) => 
            negotiations.concat(array), []))
```
O módulo `app.js` exporta apenas a função `getNegotiations()`. As demais funções são usadas internamente e para nosso problema não faz sentido serem chamadas por outro módulo que não seja o próprio módulo no qual foram declaradas. 

Excelente. Somos capazes de testar o resultado através do módulo `js/app.js`:


```javascript 
import { getNegotiations } from './api.js';

document
    .querySelector('#btn')
    .onclick = () => getNegotiations()
    .then(negotiations => {
        console.log(negotiations);
    })
    .catch(err => console.log(err.message));
```

No entanto, ainda não estamos lidando com as operações diferenciadas do evento `click` detalhadas na explicação do problema. Vamos isolá-las no módulo `js/operators.js`:


```javascript
export const debounceTime = milliseconds => {
    let timer = 0;
    return fn => () => {
        clearTimeout(timer);
        timer = setTimeout(fn, milliseconds);
    };
};
    
export const take = times => {
    let requestCounter = 0;
    return fn => () => {
        requestCounter++;
        if(requestCounter <= times) fn();
    };
}
```

A primeira função, `debounceTime()` que resolve o problema de executarmos apenas uma operação dentro de um janela de tempo. Já a segunda, `time()`, garantirá o número máximo de vezes que uma operação deve ser executada. Todavia, precisamos combinar as duas funções. 

Uma maneira elegante de combinar as operações que acabamos de criar é através de uma função especializada em composição. Inclusive, já abordei este assunto no artigo <a href="http://cangaceirojavascript.com.br/compondo-funcoes-javascript/" target="blank">"Compondo funções em JavaScript"</a>:

```javascript
export const debounceTime = milliseconds => {
    let timer = 0;
    return fn => () => {
        clearTimeout(timer);
        timer = setTimeout(fn, milliseconds);
    };
};
    
export const take = times => {
    let requestCounter = 0;
    return fn => () => {
        requestCounter++;
        if(requestCounter <= times) fn();
    };
}

// nova função!
export const compose = (...fns) => value => 
  fns.reduceRight((previousValue, fn) => 
      fn(previousValue), value);
```

Agora podemos juntar tudo e materializar nossa solução:

```javascript 
// js/app.js

import { getNegotiations } from './api.js';
import { compose, debounceTime, take } from './operators.js';

const operations = compose(debounceTime(500), take(2));

document
.querySelector('#btn')
.onclick = operations(() => 
    getNegotiations()
    .then(negotiations => {
        console.log(negotiations);
    })
    .catch(err => console.log(err));
```

## Utilizando Rxjs

Com o problema solucionado usando vanilla JavaScript, fica mais impactante demonstrar a solução através do Rxjs. Nosso código ficaria assim:

```javascript 
import { getNegotiations } from './api.js';

Rx.Observable
    .fromEvent(document.querySelector('#btn'),'click')
    .debounceTime(500)
    .take(3)
    .mergeMap(() => Rx.Observable.fromPromise(getNegotiations()))
    .subscribe(
        negociacoes => console.log(negociacoes),
        err => console.log(err)
    );
```

## Conclusão

O Rxjs vai muito além do que vimos neste artigo, porém a drástica redução da complexidade do código que escrevemos já é chamariz para aqueles interessados em investir nessa biblioteca.

E você? Utilizaria Rxjs em seus projetos? Deixe sua opnião.