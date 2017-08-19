---
layout: post
title:  "Importando módulos do Node.js com destructuring"
description: Quando desenvolvemos na plataforma Node.js é extremamente comum precisarmos importar outros módulos para realizarmos nossas tarefas.
date:   2017-08-19 08:14:00 -0300
categories:
permalink: /importando-modulos-do-nodejs-com-destructuring/
author: flavio_almeida
tags: [node, javascript, boas-praticas, destructuring]
image: logo.png
---

Quando desenvolvemos na plataforma Node.js é extremamente comum precisarmos importar outros módulos para realizarmos nossas tarefas. Vejamos um exemplo:

```javascript
// importando o módulo fs
const fs = require('fs');

fs.writeFile('arquivo.txt', 'conteúdo do arquivo', err => {

  if(err) return console.log(err);
  fs.readFile('arquivo.txt', 'utf8', (err, data) => {
  	
    if (err) return console.log(err);
    // exibe o conteúdo do arquivo
    console.log(data);
  });
}); 
```

No exemplo anterior, estamos interessados em usar as funções `writeFile` e `readFile`. Aliás, as funções foram utilizadas através da forma *error first callback*. Já vimos em <a href="http://cangaceirojavascript.com.br/por-que-voce-deveria-estar-usando-util-promisify/" target="_black">outro post</a> que podemos "promisificar" API's como essas facilmente.

Ainda no contexto da importação de módulos, podemos evitar o prefixo `fs.` para acessarmos as funções `writeFile` e `readFile` guardando-as em constantes dedicadas:

```javascript
const fs = require('fs');

// guardando em constantes
const writeFile = fs.writeFile;
const readFile = fs.readFile;

// não precisa mais do prefixo fs.
writeFile('arquivo.txt', 'conteúdo do arquivo', err => {

  if(err) return console.log(err);

  // não precisa mais do prefixo fs.
  readFile('arquivo.txt', 'utf8', (err, data) => {

    if (err) return console.log(err);
    // exibe o conteúdo do arquivo
    console.log(data);
  });
});
```

A solução anterior torna-se prática quando precisamos acessar as funções diversas vezes dentro de um módulo, no entanto podemos simplificar ainda mais a importação de módulos através de uma atribuição via desestruturação, o **destructuring** adicionando ao ECMAScript 2015 (ES6) e suportado a partir da versão 6.0 do Node.js.

## Atribuição via desestruturação (destructuring)

Vejamos o exemplo anterior usando destructuring:

```javascript
// destructuring
const { writeFile, readFile } = require('fs');

writeFile('arquivo.txt', 'conteúdo do arquivo', err => {

  if(err) return console.log(err);
  readFile('arquivo.txt', 'utf8', (err, data) => {

    if (err) return console.log(err);
    // exibe o conteúdo do arquivo
    console.log(data);
  });
});
```

Vejamos a linha:

```javascript
const { writeFile, readFile } = require('fs');
```
O retorno da chamada `require('fs')` é um objeto JavaScript com dezenas de propriedades que guardam funções. Com `destructuring`, estamos desconstruindo o objeto retornado extraindo para `writeFle` e `readFile` propriedades de mesmo nome.

Vejamos um exemplo mais simples para vermos o `destructuring` em ação:

```javascript
function exibeNome(nome) {
  console.log(nome);
}
const cangaceiro = {
  nome: 'Flávio Almeida',
  bando: 'javascript sangui no oios',
}
// extraiu para a variável nome o valor da propriedade cangaceiro.nome

const { nome } = cangaceiro;
exibeNome(nome);	
```

## Conclusão 

A atribuição via desestruturação vai além do que vimos e você pode consultar a <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Atribuicao_via_desestruturacao" target="_blank">documentação completa</a> para saber ainda mais. No entanto, a parte que vimos é suficiente para nos ajudar na importação de módulos na plataforma Node.js.
