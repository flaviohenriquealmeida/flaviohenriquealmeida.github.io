---
layout: post
title:  "Streaming de audio com Node.js"
description: Streams talvez seja um dos recursos de maior destaque na plataforma Node.js, porém ele ainda é ignorado por boa parte dos iniciantes nesta plataforma. Neste post mostrarei um exemplo bem simples e prático do seu uso fazendo uma caricatura do que acontece com aplicativos como Spotify e plataformas de audio como SoundCloud..
date:   2017-08-28 13:00:00 -0300
categories:
permalink: /streaming-audio-node/
author: flavio_almeida
tags: [javascript, streams, node]
image: logo.png
---
<a href="https://nodejs.org/api/stream.html" target="_blank">Stream</a> talvez seja um dos recursos de maior destaque na plataforma Node.js, porém ele ainda é ignorado por boa parte dos iniciantes nesta plataforma. Neste post mostrarei um exemplo bem simples e prático do seu uso fazendo uma caricatura do que acontece com aplicativos como Spotify e plataformas de audio como SoundCloud. 

Caso o leitor já tenha uma noção do conceito de `stream`, <a href="#begin">pode ir direto para a implementação do código</a> sem precisar passar pela explanação do conceito.

## Streams

Streams podem ser de leitura e de escrita. Streams de leitura podem ser associados a streams de escrita através da função `pipe` (tubo). Podemos definir o tamanho máximo em bytes a cada leitura, evitando que a memória do servidor seja sobrecarregada por carregar de uma única vez os dados da origem. 

À medida que esses dados são enviados para o destino, eles são descartados no lado do servidor e novos *chuncks* com dados são enviados até que a fonte de leitura seja exaurida. Dessa forma, temos uma previsibilidade de quanta memória o servidor utilizará, pois apenas a porção que definirmos no buffer será carregada por vez em memória. 

Veja que essa estratégia é intessante para sites como Spotify, SoundCloud, YouTube. Imaginem se todos eles já enviassem por completos os audios e os vídeos diretamente para os usuários em seus navegadores? 

Agora que já temos uma ideia básica de streams, vamos para nossa implementação.

## O projeto

Nosso projeto terá uma única página que ao ser exibida carregará um audio do servidor. A transferência do audio para `index.html` será feito através de streaming, no lugar de enviar todo arquivo de uma única vez.

<span id="begin"></span>
## Estrutura de pastas e arquivos

>*Antes de continuar, tenha certeza de estar usando Node.js 8.0 ou superior*.

Vamos criar a pasta `stream-de-audio` com a seguinte estrutura e arquivos:

```
stream-de-audio
    ├── public
    │   └── index.html
    └── server.js
```

Baixe qualquer arquivo de audio do tipo `ogg` e salve-o dentro da pasta `stream-de-audio` com o nome `audio.ogg`:

```
├── audio.ogg
├── public
│   └── index.html
└── server.js
```

## A página index.html

O conteúdo de `stream-de-audio/public/index.html` será uma simples página que possui a tag `audio` que terá como `source` o endereço `/audio` que ainda **não** existe em nosso servidor:

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Exemplo</title>
</head>
<body>
    <h1>Exemplo de Stream de Audio</h1>
    <audio controls>
        <source src="/audio" type="audio/ogg">
    </audio>
</body>
</html>
```
Agora que já temos `index.html`, podemos baixar as dependências necessárias para criar nosso servidor. 

## Baixando as dependências da aplicação

Agora, dentro da pasta `stream-de-audio` criaremos o arquivo `package.json` e instalaremos o `Express` para nos ajudar a criar nosso servidor. Para isso, basta executar dentro da pasta `stream-de-audio` os seguintes comandos na sequência:

```
npm init -y
npm i express@4.15.4 --save
```

Nossa pasta ficará assim:

```
├── audio.ogg
├── node_modules
├── package-lock.json
├── package.json
├── public
│   └── index.html
└── server.js
```

Ótimo. Agora precisamos tornar a pasta `public` acessível através do navegador.

## Compartilhando a pasta estática

Agora, em `stream-de-audio/server.js` vamos configurar o Express para compartilhar a pasta estática `public`:

```javascript
// server.js
const express = require('express');
const app = express();
app.use(express.static('public'));
app.listen(3000, () => console.log('App na porta 3000'));
```

Já podemos testar o servidor. Basta executarmos o comando `node server` dentro da pasta `stream-de-audio` para que nosso servidor fique de pé. Vamos acessar o endereço `http://localhost:3000` e verificar se o título "Exemplo de Stream de Audio" é exibido no navegador. Depois de testar, pare o servidor.

Com a certeza de que tudo esta funcionando, vamos para o próximo passo que é o endpoint que retornará o audio para a rota `/audio`, aquela usada pela tag `source` em `index.html`:

```javascript
// server.js
const express = require('express');
const app = express();
app.use(express.static('public'));

app.get('/audio', (req, res) => {
    // ainda precisamos enviar o audio
});

app.listen(3000, () => console.log('app is running'));
```

Agora que já temos nossa rota configurada, precisamos implementá-la. 

## Extraindo informações do arquivo

Queremos fazer streaming de `audio.ogg` para quem acessar o endereço `/audio`. Para isso, precisaremos da ajuda do módulo `fs` padrão do Node.

Porém, antes que possamos fazer streaming do nosso audio, precisamos saber seu tamanho em bytes, tarefa que o próprio módulo `fs` pode realizar através da função `fs.stat`. No entanto, `fs.stat` é assíncrona e para melhorarmos a legibilidade do nosso código vamos "promisificá-la" através de `util.promisify`. No post <a href="http://cangaceirojavascript.com.br/por-que-voce-deveria-estar-usando-util-promisify/" target="_blank">"Por que você deveria estar usando util promisify"</a> eu explico esse processo com detalhes. Com a API `fs.stat` promsificada, podemos usar `async/await`. Só não podemos esquecer de torna o callback de `app.get` uma função `async`:

```javascript
// server.js
const express = require('express');
const app = express();

// importou o módulo fs
const fs = require('fs');

// promisificou fs.stat
const getStat = require('util').promisify(fs.stat);

app.use(express.static('public'));

// callback é async agora!
app.get('/audio', async (req, res) => {

    const filePath = './audio.ogg';
    
    // usou a instrução await, extraiu informações do arquivo
    const stat = await getStat(filePath);
});

app.listen(3000, () => console.log('app is running'));
```

Excelente. O object `stat` possui uma série de informações sobre nosso arquivo de audio, entre elas o tamanho do arquivo.

## Adicionando informações no cabeçalho

Agora que já temos estatísticas sobre o arquivo de audio, precisamos informar seu tamanho (Content-Length) no cabeçalho de resposta, inclusive o tipo de conteúdo (Content-Type) que será enviado:

```javascript
// server.js
const express = require('express');
const app = express();
const fs = require('fs');
const getStat = require('util').promisify(fs.stat);

app.use(express.static('public'));

app.get('/audio', async (req, res) => {

    const filePath = './audio.ogg';
    
    const stat = await getStat(filePath);

    // informações sobre o tipo do conteúdo e o tamanho do arquivo
    res.writeHead(200, {
        'Content-Type': 'audio/mpeg',
        'Content-Length': stat.size
    });
});

app.listen(3000, () => console.log('app is running'));
```

Estamos quase lá. Só precisamos agora criar um *stream* de leitura para nosso arquivo de audio.

## Criando um stream através de fs.createReadStream() 

Utilizaremos agora a função `fs.createReadStream()`. Ela permite a criação de um *stream* de leitura de uma maneira muito simples. Só precisamos indicar o caminho do arquivo para começarmos a transmitir. Mas transmitir para onde, cara pálida? Para o navegador, claro. A boa notícia é que a resposta do servidor (res) também é um *stream*, no caso, um *stream* de escrita. Podemos ligar o *stream* de leitura com o de escrita através da função `pipe()`. Nosso código ficará assim:    

```javascript
// server.js
const express = require('express');
const app = express();
const fs = require('fs');
const getStat = require('util').promisify(fs.stat);

app.use(express.static('public'));

app.get('/audio', async (req, res) => {

    const filePath = './audio.ogg';
    
    const stat = await getStat(filePath);

    // informações sobre o tipo do conteúdo e o tamanho do arquivo
    res.writeHead(200, {
        'Content-Type': 'audio/mpeg',
        'Content-Length': stat.size
    });

    const stream = fs.createReadStream(filePath);

    // só exibe quando terminar de enviar tudo
    stream.on('end', () => console.log('acabou'));

    // faz streaming do audio 
    stream.pipe(res);
});

app.listen(3000, () => console.log('app is running'));
```

Vamos subir o servidor novamente com a instrução `node server` para em seguida acessarmos o endereço `http://localhost:3000`. Já podemos clicar no botão `play` para que a música seja tocada.

## Reduzindo o buffer 

No entanto, como o arquivo de audio tem um tamanho bem pequeno, não conseguiremos ver a leitura do audio de maneira gradativa pelo player enquanto o audio toca. Para que possamos ver a barra de progresso de leitura vamos diminuir drásticamente o tamanho do buffer de leitura. Isso reduzirá a quantidade de informação enviada por vez e por conseguinte fará com que a barra de progresso seja exibida.

A função `fs.createReadStream()` aceita receber um segundo parâmetro, um objeto de configuracão. Nele, adicionaremos a propriedade `highWaterMark` com o valor do buffer que desejamos utilizar:

```javascript
// server.js
const express = require('express');
const app = express();
const fs = require('fs');
const getStat = require('util').promisify(fs.stat);

app.use(express.static('public'));

// 10 * 1024 * 1024 // 10MB
// usamos um buffer minúsculo! O padrão é 64k
const highWaterMark =  2;

app.get('/audio', async (req, res) => {

    const filePath = './audio.ogg';
    
    const stat = await getStat(filePath);
    
    // informações sobre o tipo do conteúdo e o tamanho do arquivo
    res.writeHead(200, {
        'Content-Type': 'audio/mpeg',
        'Content-Length': stat.size
    });

    const stream = fs.createReadStream(filePath, { highWaterMark });

    // só exibe quando terminar de enviar tudo
    stream.on('end', () => console.log('acabou'));

    // faz streaming do audio 
    stream.pipe(res);
});

app.listen(3000, () => console.log('app is running'));
```

Parando o servidor e subindo-o novamente, já será possível ver a barra de progresso do carregamento do audio enquanto ele toca. 


