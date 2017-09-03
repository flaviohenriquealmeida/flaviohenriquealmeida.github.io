---
layout: post
title:  "Express, upload de arquivo com multer"
description: 
date:   2017-09-12 08:00:00 -0300
categories:
permalink: /express-upload-multer/
author: flavio_almeida
tags: [javascript, express, upload, node, multer]
image: logo.png
---

Realizar o upload é um processo corriqueiro nas aplicações web. Neste post mostrarei como realizar este procedimento utilizando Express e o middleware Multer.

## Estrutura de pastas e arquivos

>*Antes de continuar, tenha certeza de estar usando Node.js 8.0 ou superior*.

Vamos criar a pasta `upload-arquivo` com a seguinte estrutura e arquivos:

```
upload-arquivo
    ├── uploads
    ├── public
    │   └── index.html
    └── server.js
```

## A página index.html

O conteúdo de `upload-arquivo/public/index.html` será um formulário apontado para o endereço `file/upload` que ainda não existe em nosso servidor. 

Um ponto a destacar é a propriedade `encType` para indicar o tipo de codificação utilizado no envio dos dados, em nosso caso `multipart/form-data`. Através de um `<input type="file">` selecionamos o arquivo que desejamos enviar. Por fim, o `<input type="submit">` submeterá o formulário e por conseguinte enviará os dados do nosso arquivo:

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Express, upload de arquivos com Multer</title>
</head>
<body>   
    <h1>Express, upload de arquivos com Multer</h1>
    <form action='file/upload' method='post' encType="multipart/form-data">
        <input type="file" name="file" />
        <input type='submit' value='Upload!' />
    </form>	
</body>
</html>
```
Agora que já temos `index.html`, podemos baixar as dependências necessárias para criar nosso servidor. 

## Baixando as dependências da aplicação

Dentro da pasta `upload-arquivo` criaremos o arquivo `package.json` e instalaremos o Express e o <a href="https://github.com/expressjs/multer" target="_blanl">Multer</a> para nos ajudar a criar nosso servidor e a lidar com o upload de arquivo respectivamente. Para isso, basta executarmos dentro da pasta `upload-arquivo` a sequência de comandos:

```
npm init -y
npm i express@4.15.4 multer@1.3.0 --save
```

A pasta `upload-arquivo` estará assim: 

```
├── uploads
├── node_modules
├── package-lock.json
├── package.json
├── public
│   └── index.html
└── server.js
```

Ótimo. Agora precisamos tornar a pasta `public` acessível através do navegador para que ele consiga carregar `index.html`. 

## Compartilhando a pasta estática

Já vimos o processo de tornar uma pasta estática acessível para o navegador no post <a href="http://cangaceirojavascript.com.br/streaming-audio-node/">Streaming de audio com Node.js</a>, mas recordar é viver, não é mesmo?

Agora, em `upload-arquivo/server.js` vamos configurar o Express para compartilhar a pasta estática `public`:

```javascript
// server.js
const express = require('express')
	, app = express();

app.use(express.static('public'));
app.listen(3000, () => console.log('App na porta 3000'));
```

Já podemos testar o servidor. Basta executarmos o comando `node server` dentro da pasta `upload-arquivo` para que nosso servidor fique de pé. Vamos acessar o endereço `http://localhost:3000` e verificar se o título "Express, upload de arquivos com Multer" é exibido no navegador. Depois de testarmos, podemos parar o servidor. Ainda há mais código para ser escrito.

## Configurando multer

Agora que já temos nossa pasta estática configurada, criaremos uma instância do Multer configurada para considerar a pasta `uploads/` como pasta de destino dos uploads realizados:

```javascript
// server.js
const express = require('express')
	, app = express()
	, multer = require('multer');
    
const upload = multer({ dest: 'uploads/' });

app.use(express.static('public'));

app.listen(3000, () => console.log('App na porta 3000'));
```

A ideia agora é utilizarmos essa instância para todas as rotas que precisem lidar com o upload do arquivo. Vamos criar a rota `file/upload`:

```javascript
// server.js
const express = require('express')
	, app = express()
	, multer = require('multer');

// cria uma instância do middleware configurada
const upload = multer({ dest: 'uploads/' });

app.use(express.static('public'));

// rota indicado no atributo action do formulário
app.post('/file/upload', upload.single('file'), 
    (req, res) => res.send('<h2>Upload realizado com sucesso</h2>'));  

app.listen(3000, () => console.log('App na porta 3000'));
```

Veja que para a rota `/file/upload`, antes de lidarmos com `req` e `res`, passamos o middleware do multer como parâmetro. O `upload.sigle('file')` indica que estamos interessado no dado enviado com o `name` `file`, o mesmo name utilizado pelo `<input type="file" name="file">`.

Experimente fazer o upload de um arquivo qualquer. Você verificará que ele será salvo na pasta `upload-arquivo/uploads` com um nome diferente do `name` definido pelo `<input type="file">`, algo como `08e36ff4c9d3dc106e3a9fa2367797c9`. Se quisermos o valor do `name` precisamos criar um storage. 

## Multer Storage

Um storage nos permite um controle mais fino do processo de upload, permitindo que extraiamos o valor do `name` enviado pelo formulário através da propriedade `fieldName`:


```javascript
// server.js
const express = require('express')
	, app = express()
	, multer = require('multer');

// cria uma instância do middleware configurada
// destination: lida com o destino
// filenane: permite definir o nome do arquivo gravado
const storage = multer.diskStorage({
    destination: function (req, file, cb) {

        // error first callback
        cb(null, 'uploads/');
    },
    filename: function (req, file, cb) {

        // error first callback
        cb(null, file.fieldname + '-' + Date.now())
    }
});

// utiliza a storage para configurar a instância do multer
const upload = multer({ storage });

app.use(express.static('public'));

// continua do mesma forma 
app.post('/file/upload', upload.single('file'), 
    (req, res) => res.send('<h2>Upload realizado com sucesso</h2>'));  

app.listen(3000, () => console.log('App na porta 3000'));
```

## Salvando com a mesma extensão do arquivo

Qualquer arquivo que realizarmos o upload será salvo no disco com o nome `file-` seguido da data atual. Se quisermos manter a extensão do arquivo original podemos alterar nosso código para:

```javascript
// server.js

// não esqueça de importar o módulo path!
const express = require('express')
	, app = express()
	, multer = require('multer')
	, path = require('path');

const storage = multer.diskStorage({
    destination: function (req, file, cb) {
        cb(null, 'uploads/')
    },
    filename: function (req, file, cb) {
        cb(null, `${file.fieldname}-${Date.now()}.${path.extname(file.originalname)}`);
    }
});

// código posterior omitido 
```

Com auxílio do módulo `path` e com a propriedade `file.originalname` conseguimos extrair a extensão do arquivo. 

## Salvando com o mesmo nome do arquivo 

Pode ser que queiramos simplesmente manter o nome do arquivo original. Basta passarmos para `cb` o valor de `file.originalname` diretmente:

```javascript
// server.js
// código anterior omitido 
const storage = multer.diskStorage({
    destination: function (req, file, cb) {
        cb(null, 'uploads/')
    },
    filename: function (req, file, cb) {
        cb(null, file.originalname);
    }
});
// código posterior omitido 
```

## Conclusão

Realizar um upload na plataforma Node.js não precisa ser um martírio. Com ajuda do Express e o middlware Multer conseguimos implenentar uploads sem muito mistério.

