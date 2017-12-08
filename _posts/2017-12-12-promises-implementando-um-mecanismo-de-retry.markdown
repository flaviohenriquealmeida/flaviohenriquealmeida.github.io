---
layout: post
title:  "Promises: implementando um mecanismo de retry"
description: 
date:   2017-12-04 07:00:00 -0300
categories:
permalink: /promises-implementando-mecanismo-de-retry/
author: flavio_almeida
tags: [javascript, promise, retry, delay]
image: logo.png
---
A repercussão extremamente positiva na comunidade do meu <a href="http://cangaceirojavascript.com.br/promises-implementando-timeout-com-promise-race" target="_blank">último artigo</a> foi tamanha que aqui estou escrevendo novamente sobre Promises, desta vez sobre como implementar um mecanismo de retry. 

## O problema

Não é raro locais nos quais a internet é intermitente, inclusive em áreas com rede de alta velocidade. Acessar a internet dentro de um elevador, dentro barca ou até mesmo no estacionamento de um shopping pode contribuir para a instabilidade da rede.

Existem diferentes estratégias para se lidar com a intermitência daqui descrita, por exemplo, uma aplicação pode funcionar temporariamente offline para mais tarde sincronizar as operações do usuário ou até mesmo realizar novamente a operação um certo número de vezes dentro de um espaço de tempo para só considerar a operação fracassada depois que todas as tentativas tiverem sido esgotadas. É esta última abordagem que abordarei nesse artigo.

Temos um arquivo HTML que exibe apenas um botão e que importa o módulo `app`:

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

A ideia é a seguinte. No clique do botão, utilizando a Fetch API que adere à especificação Promise, precisamos buscar as negociações da semana, porém, se a rede ou a API consumida estiverem foras, precisaremos repetir a mesma operação no máximo três vezes, aguardando dois segundos entre as tentativas. 


Vejamos o código do módulo `app`:

```javascript
// js/app.js

/*
    Importa a função que lida com o status da requisição 
    e converte a resposta para JSON
*/
import { fetchHandler  } from './promise-util.js';
/*
    Função que ao ser chamada retorna a Promise criada pela
    API Fetch
*/
const getNegotiations = () => 
    fetch('http://localhost:3000/negociacoes/semana')
        .then(fetchHandler);
    
document
.querySelector('#btn')
.onclick = () => 
    getNegotiations()
    .then(negotiations => {
        console.log(negotiations);
        alert('Operation complete!');
    })
    .catch(err => {
        console.log(err);
        alert('Operation failed!');
    });
```

Do jeito que esta, se algum erro acontecer, a operação não será realizada novamente. A má notícia é que, na data de publicação deste artigo, Promises não suportam nativamente a capacidade de realizar novamente uma operação fracassada. Nesse sentido, teremos que implementar essa funcionalidade.

Mas antes de cairmos dentro da implementação, vamos alterar o módulo `app` para que faça uso da funcionalidade que ainda vamos criar, para termos um *big picture* de como ela funcionará:

```javascript
// js/app.js

import { fetchHandler, retry  } from './promise-util.js';

const getNegotiations = () => 
    fetch('http://localhost:3000/negociacoes/semana')
        .then(fetchHandler);
    
document
.querySelector('#btn')
.onclick = () => 
    retry(getNegotiationsTimeout, 3, 2000)
    .then(negotiations => {
        console.log(negotiations);
        alert('Operation complete!');
    })
    .catch(err => {
        console.log(err);
        alert('Operation failed!');
    });
```

Vamos lançar nosso olhar na seguinte modificação:

```javascript
retry(getNegotiationsTimeout, 3, 2000)
```
A função `retry` recebe como primeiro parâmetro uma função que ao ser invocada retornará uma Promise. Isso é importante, porque a cada tentativa precisaremos de uma nova Promise, pois segundo a específicação, uma Promise que já foi resolvida ou rejeitada não pode ir para qualquer outro estado, ou seja, ela é imutável. Em outras palavras, **na rejeição da operação precisaremos retornar uma nova Promise**.

Os demais parâmetros da função `retry` são o número de tentativas e o intervalo em milissegundos entre as tentativas. Não podemos simplesmente sair repetindo a operação, precisamos de uma folga entre as repetições.

Agora que já temos uma visão geral da chamada da função `retry`, vamos iniciar nossa implementação. Todavia, não partiremos diretamente para ela, primeiro implementaremos uma função que permita executar uma pausa (delay) entre as chamadas da Promise para só depois combiná-la com `retry`. 

## Implementando um mecanismo de delay entre chamadas de promises

Criaremos a função `delay`. Ela funcionará dessa maneira:

```javascript
// js/app.js

// função delay ainda não existe!
import { fetchHandler, delay } from './promise-util.js';

const getNegotiations = () => 
    fetch('http://localhost:3000/negociacoes/semana')
        .then(fetchHandler);
    
document
.querySelector('#btn')
.onclick = () => 
    getNegotiations()
    .then(delay(3000))
    .then(negotiations => {
        console.log(negotiations);
        alert('Operation complete!');
    })
    .catch(err => {
        console.log(err);
        alert('Operation failed!');
    });
```

Na prova de conceito acima, basta encadearmos uma chamada da função `delay` para que a próxima chamada à `then` seja postergada. A função recebe como parâmetro o tempo da espera em milissegundos. Todavia, ela deve ser capaz de receber o resultado da chamada anterior e passá-la o próximo `then()` encadeado. Sem isso, não seremos capazes de obter a lista de negociações na chamada `then(negotiations => ...)`. 

Vamos criar a função no módulo `promise-util`:

```javascript
export const fetchHandler = res => {

    if (!res.ok) throw Error('Api error!');
    return res.json();
};

// nova função 
export const delay = delay => data =>
    new Promise((resolve, reject) => 
        setTimeout(() => resolve(data), delay)
    );
```
Vamos escrutinar a função para compreendê-la melhor. Ao ser chamada, ela retorna uma nova função que recebe como parâmetro qualquer dado da chamada que antecede a chamada de `delay`. Para podermos enxergar ainda melhor, vejamos passo a passo:

```javascript
// exemplo apenas

// o dado recebido será da promise retornada por `getNegotiation()`
getNegotiations().then(delay(3000))
```
Nada nos impede de fazer isso também:

```javascript
// exemplo apenas
const meuDelay = delay(3000);
getNegotiations().then(meuDelay);
```

Nesse contexto, o resultado da promise retornada por `getNegotiations()` será passado para `meuDelay`. Fica claro agora que nossa função receberá o resultado.

Por fim, a mesma função que recebe os dados da Promise anterior, ao ser resolvida, repassará os dados recebidos para a próxima chamada encadeada à `then`, mas só depois do tempo de delay ter passado. Vejamos:


```javascript
// exemplo apenas
getNegotiations()
    .then(delay(3000))
    .then(negociacoes => console.log(negociacoes));
```

Ótimo! Agora que já aprendemos a realizar o delay entre Promises, chegou a hora de implementarmos nossa função `retry`.

## Implementando a função retry

Sabemos que nossa função `retry` deve receber uma função que ao ser chamada, retornará sempre uma nova Promise com a operação que desejamos realizar, o número de tentativas e o intervalo de tempo entre essas tentativas:

```javascript
export const retry = (fn, retries, delay) => 
```   
A primeira que que faremos é chamada a função `fn` e programar uma resposta no caso de sua rejeição, isto é, caso algum erro aconteça durante sua execução. Sabemos que lidamos com erros de Promises na função `catch`. 

A solução que utilizarei utiliza recursão. Qual um erro acontecer, chamarei novamente a função `retry` passando `fn`, o número de tentativas decrementado de um e o valor do delay. Essa chamada recursiva garantirá uma nova execução de `fn`, inclusive o decremento do número de tentativas.

O código ficará assim:


```javascript
export const retry = (fn, retries, delay) =>
    fn().catch(err => {
        console.log(retries);
        return retries > 1 
            ? retry(fn, retries - 1, delay)
            : Promise.reject(err));
    });
```    
Na cláusula `catch`, através de um `if` ternário, testamos se o número de tentativas ainda é maior do que um (esta certo, porque já gastamos uma tentativa na chamada da promise), se for, temos direito a mais uma tentativa e chamamos recursivamente `retry(fn, retries - 1, delay)`. Se o número máximo de tentativas for excedido, retornamos uma rejeição com `Promise.reject(err)` que receba a causa do último erro.

O que as chamadas recursivas farão é encadear uma sucessão de chamadas artificiais à `then`, repetindo a operação. 

Todavia, é precisa haver um intervalo entre as tentativas. Já criamos a função `delay`


```javascript
// js/promise-util.js

export const fetchHandler = res => {

    if (!res.ok) throw Error('Api error!');
    return res.json();
};
 
export const delay = delay => data =>
    new Promise((resolve, reject) => 
        setTimeout(() => resolve(data), delay)
    );

// função final
export const retry = (fn, retries, delay) =>
    fn().catch(err => {
        console.log(retries);
        return delayPromise(delay)().then(() => 
            retries > 1 
                ? retry(fn, retries - 1, delay) 
                : Promise.reject(err));
    });
```    

Foi necessário fazermos `delayPromise(delay)()`, porque `delayPromise` retorna uma função que ao ser chamada devolve uma Promise. 

O módulo `app` ficará assim:

```javascript
// js/app.js

import { fetchHandler, retry } from './promise-util.js';

const getNegotiations = () => 
    fetch('http://localhost:3000/negociacoes/semana')
        .then(fetchHandler);
    
document
.querySelector('#btn')
.onclick = () => 
    retry(getNegotiationsTimeout, 3, 2000)
    .then(negotiations => {
        console.log(negotiations);
        alert('Operation complete!');
    })
    .catch(err => {
        console.log(err);
        alert('Operation failed!');
    });
```

Podemos tornar ainda menos verbosa nossa função `retry` adotando um valor padrão para seus parâmetro `retries` e `delay`. Por fim, podemos resolver o `console.log` de `retry`, o que permitirá remover o bloco da arrow function:

```javascript

export const fetchHandler = res => {

    if (!res.ok) throw Error('Api error!');
    return res.json();
};
 
export const delay = delay => data =>
    new Promise((resolve, reject) => 
        setTimeout(() => resolve(data), delay)
    );

export const retry = (fn, retries = 3, delay = 1000) =>
    fn().catch(err => 
        delayPromise(delay)().then(() => 
            retries > 1 
                ? retry(fn, retries - 1, delay) 
                : Promise.reject(err));
    );
```    

Agora, é possível chamar `retry` dessa maneira:

```javascript
// adota os valores padrões de retries e delay
retry(getNegotiationsTimeout)
```
Mais enxuto, não?

## Conclusão 

O padrão Promise foi uma grande adição ao ES2015, porém ela carece de recursos que utilizamos no dia a dia como a repetição de uma operação no caso de um erro ou até mesmo timeout. Contudo, nada impede do programador batalhar por uma solução, melhor ainda se essa solução pode ser isolada e utilizada em outros cenários. 

E você? Achou útil esse recurso? Já precisou dele em algum momento? Deixe sua opinião.