---
layout: post
title:  "Preview de imagens antes do upload"
description: Realizar o upload de arquivos é uma tarefa corriqueira do desenvolvedor web. Em se tratando do upload de imagens, realizar o preview antes do envio pode evitar que o usuário selecione a imagem errada. Neste artigo aprenderemos como realizar o preview da imagem escolhida.
date: 2018-04-06 08:00:00 -0300
categories:
permalink: /preview-imagens-upload/
author: flavio_almeida
tags: [javascript, image, upload, preview]
image: logo.png
---

Realizar o upload de arquivos é uma tarefa corriqueira do desenvolvedor web. Em se tratando do upload de imagens, realizar o preview antes do envio pode evitar que o usuário selecione a imagem errada. Neste artigo aprenderemos como realizar o preview da imagem escolhida.

## O problema

Temos o arquivo `index.html` que define um simples `<form>` com as tags `<input type="file">` e `<input type="submit">`:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Preview</title>
</head>

<body>
    <form action='api-url-here' method='post' encType="multipart/form-data">
        <input class="file-chooser" type="file" accept="image/*">
        <input type="submit" value="upload">
    </form>  
</body>
</html>
```
Quando selecionamos uma imagem apenas seu nome será exibido ao lado do `<input type="file">`. Veremos a seguir como realizar o preview da imagem selecionada. 

## Solução 

Vamos adicionar uma tag `<img>` com a classe `preview-img` entre a escolha do arquivo e o botão de submissão. É este elemento que fará o preview da imagem selecionada:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Preview</title>
</head>

<body>
    <form action='api-url-here' method='post' encType="multipart/form-data">
        <input class="file-chooser" type="file" accept="image/*">
        <img class="preview-img">
        <input type="submit" value="upload">
    </form>
    <script>
    // nosso script fica aqui!
    </script>    
</body>
</html>
```
Para darmos continuidade à solução precisamos obter do DOM os elementos de input da seleção do arquivo e a da imagem. Para o input de seleção planejaremos uma resposta para o evento `onchange`, aquele disparado quando um arquivo qualquer for selecionado:

```javascript
const $ = document.querySelector.bind(document);
const previewImg = $('.preview-img');
const fileChooser = $('.file-chooser');
fileChooser.onchange = e => { /* falta implementar */};
```
Excelente! O evento `onchange` de um `<input type="file">` nos dá acesso ao arquivo selecionado através de `e.target.files.item(0)`:

```javascript
const $ = document.querySelector.bind(document);
const previewImg = $('.preview-I=img');
const fileChooser = $('.file-chooser');

fileChooser.onchange = e => {
    // arquivo que faremos o upload
    const fileToUpload = e.target.files.item(0);
};
```
Não podemos simplesmente atribuir `fileToUpload` à `previewImg.src`, pois o primeiro é um binário (blob). Precisamos converter o arquivo para **DataURL** que nada mais é do que a representação do binário como uma string que é automaticamente decodificada pelo navegador. Faremos essa conversão através de um `FileReader`:

```javascript
const $ = document.querySelector.bind(document);
const previewImg = $('.preview-img');
const fileChooser = $('.file-chooser');

fileChooser.onchange = e => {
    const fileToUpload = e.target.files.item(0);
    const reader = new FileReader();

    // evento disparado quando o reader terminar de ler 
    reader.onload = e => previewImg.src = e.target.result;

    // solicita ao reader que leia o arquivo 
    // transformando-o para DataURL. 
    // Isso disparará o evento reader.onload.
    reader.readAsDataURL(fileToUpload);
};
```
Quando `reader.readAsDataURL` terminar, o evento `reader.onload` será disparado. Através do parâmetro do evento temos acesso ao resultado da conversão através de `e.target.result`. É importante perceber que esta é uma operação assíncrona.

Pronto! Ao selecionarmos uma imagem ela será exibida para o usuário permitindo-o verificar se ela é a que ele realmente deseja subir.

## Bônus: alterando a label do botão de seleção de arquivo 

Uma coisa bem incômoda do `<input type="file">` é que ele não permite mudar o texto do botão. Inclusive quem utiliza <a href="https://getbootstrap.com/" target="_blank">Bootstrap</a> pode ter alguma dificuldade em estilizá-lo.

Uma das soluções é escondermos o `<input type="file">` e dispará-lo através do clique de um `<input type="button">`. Vejamos a solução.

Primeiro, vamos adicionar o `<input type="button">` e tornar o `<input type="file">` invisível através do atributo `hidden`:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Preview</title>
</head>

<body>
    <form action='image/upload' method='post' encType="multipart/form-data">
        
        <!-- novo elemento! -->
        <input class="file-button" type="button" value="Choose your beautiful image">

        <!-- invisível -->

        <input class="file-chooser" type="file" accept="image/*" hidden>
        <img class="preview-img">
        <input type="submit" value="upload">
    </form>
    <script>
    // nosso script fica aqui!
    </script>    
</body>
</html>
```
Agora, vamos obter o novo botão para em seguida adicionarmos o evento `onclick`. É neste evento que chamaremos `fileChooser.click()`, forçando assim o click e como resultado a seleção do arquivo:

```javascript
const $ = document.querySelector.bind(document);

const previewImg = $('.preview-img');
const fileChooser = $('.file-chooser');
const fileButton = $('.file-button');

fileButton.onclick = () => fileChooser.click();

fileChooser.onchange = e => {
    const fileToUpload = e.target.files.item(0);
    const reader = new FileReader();
    reader.onload = e => previewImg.src = e.target.result;
    reader.readAsDataURL(fileToUpload);
};
```

Perfeito. Agora temos maior liberdade para estilizar nosso botão.

## Conclusão

Muitas vezes a tecnologia vigente não traz uma resposta direta para determinado problema de usabilidade e com isso força o desenvolvedor a elaborar soluções utilizando os recursos que tem. E você? Já precisou realizar o preview de imagens antes? Deixe sua opinião.
