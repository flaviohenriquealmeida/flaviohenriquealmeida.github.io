---
layout: post
title:  "Streams standard, o poder de streams agora nos navegadores"
description: Até bem pouco tempo não havia uma especificação de streams para os navegadores (...) a WHATWG criou uma especificação de streams como Web API nos browser.
date:   2017-10-25 08:00:00 -0300
categories:
permalink: /streams-standard-poder-streams-agora-navegadores/
author: flavio_almeida
tags: [streams, browser, web, api, javascript, standard, api, navegador]
image: logo.png
---

Sem dúvidas, streams é (no singular mesmo) um recurso  extremamente poderoso na plataforma Node.js. Sua API é totalmente padronizada nesta plataforma. Inclusive já vimos este poderoso recurso no artigo <a href="http://cangaceirojavascript.com.br/streaming-audio-node/" target="_blank">Streaming de audio com Node.js</a>. Todavia, até bem pouco tempo, não havia uma especificação de streams para os navegadores do mercado, apesar de muitos desenvolvedores já lidarem indiretamente com este recurso através de <a href="https://pt.wikipedia.org/wiki/WebSocket" target="_blank">WebSocket</a> com ou sem auxílio de bibliotecas como <a href="https://socket.io/" target="_blank">Socket.io</a>. 

A boa notícia é que a <a href="https://whatwg.org/">WHATWG</a> criou uma <a href="https://streams.spec.whatwg.org/" target="_blank">especificação de streams como Web API nos browser</a>, inclusive já podemos utilizá-la em alguns navegadores do mercado. É apenas uma questão de tempo até que a API se consolide.

Este post visa dar uma visão geral de como essa API pode ajudar os desenvolvedores a criarem aplicações mais responsivas e com menos consumo de memória.

## Um problema

Temos um exemplo hipotético no qual precisaremos consumir uma API que retornará um array de negociações de bolsa de valores no formato JSON:

```javascript
/* 
    Tratamento de erro omitido para focarmos 
    no essencial deste artigo.
*/
(async () => {

    const res = await fetch('http://endereco-da-api.com/negociacoes/semana');

    if(res.ok) res.json()
        .forEach(negociacao => 
            console.log(negociacao));
})();
```

Excelente. Realizamos uma requisição assíncrona através da Fetch API e com o auxílio da função `res.json()` realizamos o parser da resposta. Tudo uma maravilha, no entanto, é retornada uma lista com 7.000 objetos que representam negociações! Nesse contexto, `res.json()` só será chamado assim que toda a resposta tiver sido retornada e, quando for chamado, terá que realizar o parser de 7.000 objetos de uma só vez. Esses detalhes são suficientes para deixar nossa aplicação menos responsiva. De quebra, após o parser ser realizado, teremos 7.000 objetos em memória que serão iterados e exibidos no console. Além do problema da responsividade, temos também o problema do uso exacerbado de memória. 

Agora que já temos o *big picture* do problema, veremos como a API de streams pode nos ajudar a resolvê-lo.

## Uma solução com streams

>*Você pode testar o código a seguir em seu navegador se assim desejar (precisará de uma API). O autor utilizou a versão 61 do Google Chrome.*

Vejamos o mesmo código anterior, só que dessa vez utilizando streams. Em seguida, escrutinaremos cada parte do código:


```javascript
(async () => {

    const decoder = new TextDecoder();
    const res = await fetch('http://endereco-da-api.com/negociacoes/semana');

    if(res.ok) {
        
        const reader = res.body.getReader();

        while(!(chunk= await reader.read()).done) {
            const negociacao = JSON.parse(decoder.decode(chunk.value));
            console.log(negociacoes);
        }
    }
})()    

```

Continuamos utilizando a Fetch API, mas desta vez não obtemos a resposta no formato JSON através de `res.json()`. Vejamos o que foi feito. 

## ReadableStream

Através de `res.body.getReader()` obtemos um **ReadableStream** que armazenamos na variável `reader`. Isso é possível porque a Fetch API adere à especificação de streams. Por fim, através da chamada de `reader.read()`, que retorna uma Promise, recebemos um **chunk**. 

Antes de continuarmos, vale ressaltar que ao longo de todo exemplo foi utilizado <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Statements/funcoes_assincronas" target="_blank">async/wait</a> para tornar ainda mais legível o código.  Enfim, mas o que seria esse chunk? Primo do "Chunk" Norris?

## Chunk

Um chunk nada mais é do que um pedacinho da resposta. Ele possui duas propriedades notáveis, a `chunk.value` e a `chunk.done`. Na primeira temos acesso à resposta bruta vinda do servidor, já na segunda temos uma propriedade de controle para nos indicar quando o stream for totalmente consumido, isto é, quando `chunk.done` for `false`. 

Cada chunk será um objeto do array de negociações retornados, porém como a resposta vem em um fluxo de dados, é necessário decodificá-la primeiro para então realizarmos seu parse. 

É ai que entra o `TextDecoder` que criamos logo no início do programa e que foi armazenado na variável `decoder`. 

## TextDecoder

Através do método `decoder.decode` do TextDecoder passamos o `chunk.value`. Depois de decodificado, usamos o clássico `JSON.parse()` para convertemos a resposta para o formato JSON. Percebam que desta forma, não temos carregado de uma única vez todas as negociações em memória, pelo contrário, lidamos com a resposta pedacinho por pedacinho, evitando assim que uma grande quantidade de memória seja utilizada, além de evitarmos o bloqueio da thread principal que processa a UI e os eventos disparados pelo usuário.

## Suporte

Na data de publicação deste artigo apenas o Chrome 52 e o Opera 39 em diante suportam streams. Todavia, recursos mais sofisticados como transformações através de pipes foram suportados apenas a partir das versões 59 e 46 desses navegadores.

## Conclusão

Lidar com Streams não é novidade na plataforma Node.js, mas pela primeira vez uma Web API para o browser foi especificada. Promisora, essa especificação pode servir de guia para bibliotecas e frameworks do mercado. Quando isso acontecerá? Só o tempo dirá!

E você? Consegue enxergar outras vantagens do uso de streams no browser?