---
layout: post
title:  "Pipeline operator, uma proposta interessante ao tc39"
description: Há algum tempo foi proposto ao tc39, o comitê que participa da evolução do JavaScript, o pipeline operator. Essa feature, fortemente inspirada em linguagens como Elixir e F#, introduz o novo operador |>.
date:   2017-10-23 08:00:00 -0300
categories:
permalink: /pipeline-operator-proposta-interessante-tc39/
author: flavio_almeida
tags: [pipeline operator, piping, function composition, javascript]
image: logo.png
---

Há algum tempo foi proposto ao tc39, o comitê que participa da evolução do JavaScript, o <a href="https://github.com/tc39/proposal-pipeline-operator" target="_blank">pipeline operator</a>. Essa *feature*, fortemente inspirada em linguagens como <a href="https://elixirschool.com/pt/lessons/basics/pipe-operator/" target="_blank">Elixir</a> e <a href="https://stackoverflow.com/questions/12921197/why-does-the-pipe-operator-work" target="_blank">F#</a>, introduz o novo operador `|>`. 

Para que possamos entender como este operador mudará a estética do código que escrevemos hoje, nada melhor do que um exemplo com e sem o uso do pipeline operator. Aliás, aproveitarei o mesmo exemplo do artigo <a href="http://cangaceirojavascript.com.br/compondo-funcoes-javascript/" target="_blank">"Compondo funções em JavaScript"</a> publicado por este mesmo autor:

```javascript
const trim = text => text.trim();
const toUpperCase = text => text.toUpperCase();
const split = separator => text => text.split(separator);

const word = ' Calopsita do Agreste ';

// sem o pipeline operator
const words = 
  split(' ')( 
    toUpperCase(
      trim(word)
    )
  );

console.log(words); // ['CALOPSITA', 'DO', 'AGRESTE']
```
Temos o encadeamento das funções `split`, `toUpperCase` e `trim` para processar a string `' Calopsita do Agreste  '`. 

## Pipeline operator em ação

Agora, vejamos as mesmas operações, dessa vez com o pipeline operator:

```javascript
const trim = text => text.trim();
const toUpperCase = text => text.toUpperCase();
const split = separator => text => text.split(separator);
const word = ' Calopsita do Agreste '

// com o pipeline operator
const words =  word 
  |> trim 
  |> toUpperCase 
  |> split(' ');
  
console.log(words); // ['CALOPSITA', 'DO', 'AGRESTE']
```

Agora, fica bem claro para quem esta lendo que a primeira transformação a ser realizada é através da função `trim` e assim por diante.

## Como utilizar o pipeline operator hoje

Para quem não deseja esperar o trâmite padrão da proposta no tc39 e já deseja ter um gostinho do pipeline operador pode lançar mão do <a href="https://www.npmjs.com/package/babel-plugin-transform-pipeline" target="_blank">babel-plugin-transform-pipeline</a>, um plugin do <a href="https://babeljs.io/"  target="_blank">Babel</a>. Todavia esse autor teve dificuldades em fazê-lo funcionar, pois uma atualização do <a href="https://github.com/babel/babylon" target="_blank">babylon</a>, o parser JavaScript do Babel, <a href="https://github.com/SuperPaintman/babel-plugin-transform-pipeline/issues/1" target="_blank">quebrou recentemente o plugin</a>.

## Conclusão

O pipeline operator, ainda em estágio embrionário, promete melhorar a manutenção e a legibilidade de um código escrito principalmente no paradigma funcional. E você? Utilizaria esse operador? Deixe sua opinião.