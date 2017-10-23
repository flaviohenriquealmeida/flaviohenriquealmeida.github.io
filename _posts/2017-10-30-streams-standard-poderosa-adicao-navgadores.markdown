---
layout: post
title:  "Streams standard, uma poderosa adição aos navegadores"
description: 
date:   2017-10-23 08:00:00 -0300
categories:
permalink: /streams-standard-poderosa-adicao-navegadores/
author: flavio_almeida
tags: [streams, browser, web, api, javascript, standard, api, navegador]
image: logo.png
---

Sem dúvidas, streams é (no singular mesmo) um recurso  extremamente poderoso na plataforma Node.js. Sua API é totalmente padronizada nesta plataforma. Inclusive já vimos este poderoso recurso no artigo <a href="http://cangaceirojavascript.com.br/streaming-audio-node/" target="_blank">Streaming de audio com Node.js</a>. Todavia, até bem pouco tempo, não havia uma especificação de streams para os navegadores do mercado, apesar de muitos desenvolvedores já lidarem indiretamente com este recurso através de <a href="https://pt.wikipedia.org/wiki/WebSocket" target="_blank">WebSocket</a> com auxílio da biblioteca <a href="https://socket.io/" target="_blank">Socket.io</a>. 

A boa notícia é que a <a href="https://whatwg.org/">WHATWG</a> criou uma <a href="https://streams.spec.whatwg.org/" target="_blank">especificação de streams como Web API nos browser</a>, inclusive já podemos utilizá-la em alguns navegadores do mercado. É apenas uma questão de tempo até que a API se consolide.

Este post visa dar uma visão geral de como essa API pode ajudar os desenvolvedores a criarem aplicações mais responsivas e com menos consumo de memória.

## Um problema

Temos um exemplo hipotético no qual precisaremos consumir uma API que retornará um array de negociações de bolsa de valores no formato JSON:

```javascript
(async () => {
    /* 
        Tratamento de erro omitido 
        para focarmos no essencial
    */
    const res = await fetch('http://endereco-da-api.com/negociacoes/semana');

    if(res.ok) res.json()
        .forEach(negociacao => 
            console.log(negociacao));
})();
```

Excelente. Realizamos uma requisição assíncrona através da Fetch API e com o auxílio da função `res.json()` realizamos o parser da resposta. Tudo uma maravilha, no entanto, é retornada uma lista com 20.000 objetos que representam negociações. Nesse contexto, `res.json()` só será chamado assim que toda a resposta tiver sido retornada, o que pode tornar nossa aplicação menos responsiva. De quebra, após o parser ser realizado, teremos 20.000 objetos em memória que serão iterados e exibidos no console. Além do problema da responsividade, temos também o problema do uso exacerbado de memória. 

Agora que já temos o *big picture* do problema, veremos a seguir como a API de streams pode nos ajudar a resolvê-lo.

## Uma solução com streams

>*O código desta seção foi testado no Chrome 61.*

Vejamos o mesmo código anterior, só que dessa vez utilizando streams. Em seguida, escrutinaremos cada parte do código:


```javascript
(async () => {

    const decoder = new TextDecoder();
    const res = await fetch('http://endereco-da-api.com/negociacoes/semana');

    if(res.ok) {

        /*
            A API Fetch adere à especificação de streams. 
            O retorno de res.body.getReader() é um 
            ReadableStream!
        */
        const reader = res.body.getReader();

        /*
            A função `reader.read()` é assíncrona devolvendo
            uma promise. 

            Iterando em cada pedacinho (chunk) retornado
        */

        while(!(chunk= await reader.read()).done) {
            const negociacao = JSON.parse(decoder.decode(chunk.value));
            console.log(negociacoes);
        }
    }
})()    

```

Continuamos utilizando a Fetch API, mas desta vez não obtemos a resposta no formato JSON através de `res.json()`. 

## ReadableStream

Através de `res.body.getReader()` obtemos um **ReadableStream** que armazenamos na variável `reader`. É através de `reader.read()`, uma operação assíncrona que retorna uma Promise que por sua vez nos dá acesso a um **chunk**. Ao longo de todo exemplo foi utilizado <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Statements/funcoes_assincronas" target="_blank">async/wait</a> para tornar ainda mais legível o código.

## Chunk

Um chunk nada mais é do que um pedacinho da resposta. Ele possui duas propriedades notáveis, a `chunk.value` e a `chunk.done`. Na primeira temos acesso à resposta bruta vinda do servidor, já na segunda temos uma propriedade de controle para nos indicar quando o stream for totalmente consumido, isto é, quando `chunk.done` for `false`. 

Como podemos ver, a Fetch API adere à especificação de Streams, cada chunk será um objeto do array de negociações retornados, porém como a resposta vem em um fluxo de dados, é necessário decodificá-la primeiro para então realizarmos seu parse. 

É ai que entra o `TextDecoder` que criamos logo no início do programa e que foi armazenado na variável `decoder`. 

## TextDecoder

Através do método `decoder.decode` do TextDecoder passamos o `chunk.value`. Depois de decodificado, usamos o clássico `JSON.parse()` para convertemos a resposta para o formato JSON. Percebam que desta forma, não temos carregado de uma única vez todas as negociações em memória, pelo contrário, lidamos com a resposta pedacinho por pedacinho, evitando assim que uma grande quantidade de memória seja utilizada, além de evitarmos o bloqueio da thread principal que processa a UI e os eventos disparados pelo usuário.

## Conclusão

Lidar com Streams não é novidade na plataforma Node.js, mas pela primeira vez uma Web API para o browser foi especificada. Promisora, essa especificação pode servir de guia para bibliotecas e frameworks do mercado. Quando isso acontecerá? Só o tempo dirá!

E você? Consegue enxergar outras vantagens do uso de streams no browser?