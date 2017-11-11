---
layout: post
title:  "Importação nativa de módulos no browser"
description:
date:   2017-11-11 07:00:00 -0300
categories:
permalink: /importacao-nativa-modulos-browser/
author: flavio_almeida
tags: [ESM, loader, ES6, ES2015, script, module, javascript, browser]
image: logo.png
---
A especificação ES2015 (ES6) introduziu um sistema de módulos baseado na sintaxe `import` e `export`, porém não definiu como o carregamento e a resolução de módulos deveriam ser implementados por falta de consenso entre os fabricantes de browsers. Essa situação abriu espaço para que a comunidade criasse carregadores de módulos, os famosos `loaders`. 

A boa notícia é que os browsers já começaram a implementar o suporte ao carregamento nativo de ESM (ECMASCRIPT modules). Neste post veremos como utilizar este recurso que esta sendo consolidado.  

Você pode pular, se preferir, a explicação do problema que o sistema de módulos veio resolver e ir direto para a importação nativa de módulos indo direto para <a href="#typeModule">esta seção</a>.

## O problema

Temos um projeto com a seguinte estrutura:

```bash
├── index.html
└── js
    └── script-c.js  
    └── script-b.js
    ├── script-a.js
```

O `script-a.js` depende da função `sumTreeMumbers` declarada no `script-b.js` que por sua vez depende da função `sumTwoNumbers` declarada no módulo `script-c.js`. Vejamos cada um dos scripts:

*script-c.js, não depende de ninguém:*
```javascript
const sumTwoNumbers = (number1, number2) => number1 + numer2;
```

*script-b.js, depende de script-c.js*
```javascript
const sumThreeNumbers = (mumber1, number2, number3) => 
    sumTwoNumber(number1, number2) + number3;
```

*script-a.js, depende de script-b.js*
```javascript
const total = sumTreeNumbers(10, 20, 30);
console.log(total);
```

Precisaremos importar os três scripts em `index.html` e obrigatoriamente seguir a ordem de dependência, caso contrário teremos um erro:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Exemplo</title>
</head>
<body>
    <script src="js/script-c.js"></script>
    <script src="js/script-b.js"></script>
    <script src="js/script-a.js"></script>
</body>
</html>
```

Temos apenas três scripts e não teríamos problemas de verificar se cada script da árvore de dependências esta presente em `index.html`, inclusive garantir a ordem de importação. Todavia, em um projeto mais complexo, isso pode se tornar um pesadelo. Por fim, todas as funções declaradas estão no escopo global. Faz sentido que nosso `script-a.js` enxergue a função declarada em `script-b.js`, mas não aquela declarada em `script-c.js`. 

Para resolver todos esses problemas que acabamos de listar, vamos transformar cada script em um módulo do ES2015 (ES6) e utilizar as instruções `import` e `export`. 

## Módulo e as instruções import e export

Quando trabalhamos com o sistema de módulos do ES2015 cada script por padrão é um módulo que confina tudo o que possui, em outras palavras, nenhuma variável ou função escapará para o escopo global. Apenas os artefatos explicitamente exportados com a instrução `export` poderão ser importados por outros módulos através da instrução `import`.

Modificando nosso código:

*script-c.js:*
```javascript
export const sumTwoNumbers = (number1, number2) => number1 + number2;
```

*script-b.js:*
```javascript
import { sumTwoNumbers } from './script-c.js';

export const sumThreeNumbers = (number1, number2, number3) => 
    sumTwoNumbers(number1, number2) + number3;
```

*script-a.js:*
```javascript
import { sumThreeNumbers } from './script-b.js';

const total = sumThreeNumbers(10, 20, 30);
alert(total);
```

Excelente, conseguimos resolver o problema do escopo global fazendo cada módulo importar explicitamente o que precisa de outro. Todavia, como faremos o carregamento desses módulos no navegador? Não podemoss implesmente importada cada módulo em separado como no exemplo anterior, vejamos:

```html
<!-- não rola fazer isso! -->
<script src="js/script-c.js"></script>
<script src="js/script-b.js"></script>
<script src="js/script-a.js"></script>
```

O navegador não entenderá que cada script é na verdade um módulo e não reconhecerá a sintaxe `import`, muito menos `export`. Além disso, importar cada módulo que desejamos utilizar no navegador também é um problema, pois teremos que saber quais são todos os scripts que `script-a.js` depende. Já imaginaram isso com uma árvore de dependência muito maior? Seríamos obrigados a ter tudo explicado em um documento, aquele que será abandonado com o tempo e que não refletirá mais a realidade do sistema. 

A especificação ES2015 deixou em aberto como o carregamento dos módulos deve ser feita, abrindo espaço para que a comunidade criasse carregadores de módulos, os famosos *loaders* (scripts especializados no carregamento de módulos) para realizar essa tarefa. A boa notícia é que os fabricantes de browsers parecem ter chegado a um consenso e hoje podemos carregar módulos nativamente em boa parte dos navegadores.

<span id="typeModule"></span>
## Carregando módulos nativamente nos navegadores. 

>*O autor testou o código a seguir no Chrome 62.0.3202.75 e no Safari Versão 11.0.1 sem problema algum. Porém, no Firefox (56.0.2) e no Edge é necessário ativar flags experimentais para que o recurso funcione.*

Para carregarmos um módulo no navegador continuamos utilizando a tag `<script>`, mas adicionando o atributo `type="module"`. Vejamos:

```html
<!-- único script apenas -->
<script src="js/script-a.js" type="module"></script>
```

Veja que além de usarmos `<script type="module">` importamos apenas um único módulo, o módulo ponto de entrada da aplicação. A partir dele todas as dependências da aplicação serão resolvidas sem que o programador tenha que se preocupar com a ordem de carregamento. **Mas atenção**, essa importação só funcionará se estivermos servindo nossa página através de um servidor web, pois a busca dos módulos serão realizadas através do JavaScript, isto é, através de requisições Ajax. Outra ponto é que você não pode remover a extensão `.js` do nome do módulo importado com a instrução `import`. 

## Conclusão

O suporte ao carregamento nativo de módulos nos navegadores veio tornar mais acessível esse fantástico recursos para aplicações corriqueiras e que não utilizam frameworks ou sistemas de empacotamento de módulos como  <a href="https://webpack.github.io/" target="_blank">Webpack</a> entre outros.




