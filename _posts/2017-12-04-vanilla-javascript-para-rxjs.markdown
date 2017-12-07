---
layout: post
title:  "De vanilla JavaScript para RxJS"
description: Essa semana um dos estagiários aqui da Caelum me perguntou se valia a pena investir no estudo do Rxjs. Rxjs é uma biblioteca JavaScript que traz o conceito de programação reativa para a Web. Todavia, nada nos impede de utilizá-la na plataforma Node.js através do rx-node.
date:   2017-12-04 07:00:00 -0300
categories:
permalink: /de-vanilla-javascript-para-rxjs/
author: flavio_almeida
tags: [javascript, rxjs, functional, reactive]
image: logo.png
---

Essa semana um dos estagiários aqui da <a href="https://www.caelum.com.br/" target="_blank">Caelum</a> me perguntou se valia a pena investir no estudo do **Rxjs**. <a href="https://github.com/Reactive-Extensions/RxJS" target="_blank">Rxjs</a> é uma biblioteca JavaScript que traz o conceito de programação reactiva para a Web. Todavia, nada nos impede de utilizá-la na plataforma Node.js através do <a href="https://github.com/Reactive-Extensions/rx-node" target="_blank">rx-node</a>.

 Minha resposta poderia ser simplesmente um "sim" ou um "não". Todavia, não poderia dar uma resposta tão direta, pois ela não teria efeito transformador em seu conhecimento. 
 
 Eu também não queria entrar em detalhes técnicos e filosóficos da programação reativa, eu precisava de algo mais concreto. Foi então que decidi expor uma problema para solucionarmos com vanilla JavaScript para no final, demonstrar o mesmo código utilizando Rxjs. Nosso estagiário curtiu a ideia, até porque, o código não mente, ou ele funciona ou não funciona.

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

>*O código anterior utiliza importação nativa de módulos do ES2015. Você pode consultar o artigo  <a href="http://cangaceirojavascript.com.br/importacao-nativa-modulos-browser/" target="_blank">"Importação nativa de módulos no browser"</a> deste mesmo autor para saber.*

Quando o usuário clicar no botão, precisaremos buscar de uma API as negociações da semana atual e da semana anterior exibindo o resultado no console. Porém, se o usuário clicar mais de uma vez no botão dentro de uma janela de tempo de meio segundo, independente da quantidade de cliques realizados, apenas um deverá ser processado. Por fim, ele só poderá realizar a busca três vezes e qualquer tentativa de realizar uma nova busca acima desse limite deve ser ignorada. 

## Solucionando com vanilla JavaScript

Primeiro, vamos criar o arquivo `js/api.js`:

```javascript
// js/api.js

const api = 'http://localhost:3000';

// função utilitária para lidar com o status da requisição e conversão
const fetchHandler = res => {
  if(!res.ok) throw Error(res.statusText);
  return res.json();
};
// busca as negociações da semana 
const getNegotiationsFromWeek  = () =>
    fetch(`${api}/negociacoes/semana`)
    .then(fetchHandler)
    .catch(err => {
        console.log(err);   
        return Promise.reject('getNegotiationsFromWeek: failure!');
    });
// busca as negociações da semana anterior
const getNegotiationsFromPreviousWeek = () =>
    fetch(`${api}/negociacoes/anterior`)
    .then(fetchHandler)
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
    .then(negotiations => console.log(negotiations)
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
};
```
>*Sobre minha implementação do debounce pattern aqui apresentado, o leitor pode consultar meu post <a href="http://blog.alura.com.br/javascript-debounce-pattern-closure-e-duas-amigas/" target="_blank">"JavaScript, debounce pattern, closure e duas amigas"</a> publicado no blog da AluraOnline. Inclusive, a função `take` segue estrutura similar* 

A primeira função, `debounceTime`, é aquela que resolve o problema de executarmos apenas uma operação dentro de um janela de tempo. Já a segunda, `take`, garantirá o número máximo de vezes que uma operação deve ser executada.

## Entendendo a estrutura das operações.

Vamos escolher a função `take()` para demonstrar seu uso isoladamente primeiro:

```javascript
/* 
    A função take() retorna uma função configurada para 
    executar outra no máximo 3 vezes. Através de closure, ela 
    lembrará do parâmetro passado para take, no caso, 3
*/
const configuredTake = take(3);
/* 
    A configuredTake recebe como parâmetro a lógica que desejamos 
    executar, ou seja, uma função. Por fim, ela retorna outra função 
    que ao ser chamada, executará o callback passado para configuredTake
*/
const fn = configuredTake(() => alert('hi'));

fn(); // exibe o alerta
fn(); // exibe o alerta
fn(); // exibe o alerta
fn(); // não exibe o alerta
```

Excelente, mas precisamos combinar as funções `debounceTime` e `take`.

## Compondo funções

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
};
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

const operations = compose(debounceTime(500), take(3));

document
.querySelector('#btn')
.onclick = operations(() => 
    getNegotiations()
    .then(negotiations => console.log(negotiations))
    .catch(err => console.log(err))
);
```

Outra opção é lançarmos mão do *async/await*. Nosso código ficaria assim:

```javascript
document
.querySelector('#btn')
.onclick = operations(async () => {
    try {
    	const negotiations = await getNegotiations();
    	console.log(negotiations);
    } catch(err) {
    	err => console.log(err);
    }
}); 
```

Mas como seria nosso código utilizando Rxjs? É o que veremos a seguir.

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

Não houve a necessidade de todo o código escrito no módulo `js/operators.js`! O mais importante foi o estagiário sentir na pele o antes e o depois. Agora que ele visualmente já viu uma melhoria no código, eu posso dar início a uma explicação detalhada do que foi feito. Aliás, assunto interessante para um próximo artigo aqui no blog.

## Conclusão

O Rxjs vai muito além do que vimos neste artigo, porém a drástica redução da complexidade do código que escrevemos já é chamariz para aqueles interessados em investir nessa biblioteca.

E você? Utilizaria Rxjs em seus projetos? Deixe sua opinião.