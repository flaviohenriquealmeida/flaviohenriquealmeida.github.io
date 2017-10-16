---
layout: post
title:  "JavaScript e o ponto e vírgula, uma polêmica que atravessa o tempo"
description: Utilizar ponto e vírgula ou não, eis a questão! A polêmica sobre o assunto é antiga. Seu ápice foi em 2002 com o artigo The Infernal Semicolon ("Ponto e Vírgula dos Infernos") publicado por Bredan Eich, pai do JavaScript, e com a reposta de Douglas Crockford, autor do livro JavaScript, the Good Parts, no Github do Bootstrap, chegando a alegar que a omissão do ponto e vírgula tornaria nosso código um "insanely stupid code".  
date:   2017-10-16 15:00:00 -0300
categories:
permalink: /javascript-ponto-virgula-polemica-atravessa-tempo/
author: flavio_almeida
tags: [javascript, semicolon, ponto e vírgula, boas práticas]
image: logo.png
---

## Uma polêmica antiga

Utilizar ponto e vírgula ou não, eis a questão! A polêmica sobre o assunto é antiga. Seu ápice foi em 2002 com o artigo  <a href="https://brendaneich.com/2012/04/the-infernal-semicolon" target="_blank">The Infernal Semicolon</a> ("Ponto e Vírgula dos Infernos") publicado por Bredan Eich, pai do JavaScript, e com a reposta de Douglas Crockford, autor do livro JavaScript: *the Good Parts*<a href="https://github.com/twbs/bootstrap/issues/3057" target="_blank">, no Github do Bootstrap</a>, chegando a alegar que a omissão do ponto e vírgula tornaria nosso código um "*insanely stupid code*". Mas qual era o contexto que foi o estopim para a discussão do uso ou não do ponto e vírgula? Aliás, utilizarei a palavra em inglês **semicolon** no lugar "ponto e vírgula" devido à brevidade do primeiro ao longo deste artigo.

## O contexto primevo

 Bredan Eich em seu post alega que a omissão do semicolon quebraria minificadores JavaScript como  <a href="https://www.crockford.com/javascript/jsmin.html" target="_blank">JSMin</a>, que não realizavam ASI durante o processo de minificação. Crockford, autor do JSMin,  não achava que essa era a responsabilidade de um minificador e por isso não teve interesse em implementar essa funcionalidade, inclusive Brendon Eich foi solidário ao colega respeitando sua posição. Mas o que seria esse tal de ASI?

## ASI (Automatic Semicolon Insertion)

O ASI (Automatic Semicolon Insertion) consiste na inserção compulsória e automática do semicolon a partir de <a href="https://www.ecma-international.org/ecma-262/5.1/#sec-7.9" target="_blank">regras definidas na especificação ECMASCRIPT</a> durante o parse do script pelo interpretador. **Para o interpretador do JavaScript, um código sem semicolon é um código inválido**. Nesse sentido, fica notória a aplicação do ASI independente se escrevemos um código com ou sem semicolon. Segundo o criador da linguagem JavaScript, o <a href="https://brendaneich.com/2012/04/the-infernal-semicolon" target="_blank">ASI nada mais é do que um procedimento de correção de erro sintático</a>:

> ASI is (formally speaking) a syntactic error correction procedure. --Bredan Eich

Ainda no contexto primevo da discussão em 2002, para que minificadores pudessem processar scripts sem semicolon, era necessário que aplicassem as mesmas regras do ASI antes de processá-los, pois a transformação inline de um código sem semicolon feriria as regras do ASI resultando em um código inválido. 

Muitos desenvolvedores defendiam que implementar a lógica do ASI não era responsabilidade dos minificadores, inclusive essa era a posição adotada por Crockford como já vimos. Porém, havia quem defendesse sua implementação, justamente pelo ASI fazer parte da especificação ECMASCRIPT. 

O mais interessante é que o ASI pode pregar uma peça tanto para os que utilizam o semicolon quanto para os que o omitem. Que tal uma breve pausa para vermos essa questão?

## Intermission: quando o resultado não é o esperado

Temos a função `numberParaReal` que converte um número para a moeda brasileira. Como podemos ver, o semicolon foi omitido:


```javascript
// sem semicolon
function numberParaReal(number) {

    return 
        `R$ ${number.toFixed(2).replace('.', ',')}`
}

const real = numberParaReal(10.20) 
```

Será impresso no console `R$ 10,20` ou `undefined`? Se você escolheu `undefined` acertou! Depois de processado pelo ASI, o seguinte código ficará assim:

```javascript
// código após ASI ser aplicado
function numberParaReal(number) {

    return; // semicolon adicionado pelo ASI
    `R$ ${number.toFixed(2).replace('.', ',')}`; // semicolon adicionado pelo ASI
}

const real = numberParaReal(10.20); // o resultado é undefined
```

Uma das regras que o ASI define é que toda declaração deve terminar com ponto e vírgula. Como o valor retornado pelo `return` foi quebrado em outra linha, para o ASI são duas instruções!

E se alterarmos nosso código original para usar o semicolon no final? Vejamos:


```javascript
// com semicolon
function numberParaReal(number) {

    return 
        `R$ ${number.toFixed(2).replace('.', ',')}`;
}

const real = numberParaReal(10.20); 
console.log(real); // undefined
```

O resultado continuará sendo `undefined`, pois o resultado da aplicação do ASI continuará sendo:

```javascript
function numberParaReal(number) {

    return; // adicionando pelo ASI
        `R$ ${number.toFixed(2).replace('.', ',')}`;
}

const real = numberParaReal(10.20); 
console.log(real); // undefined
```
Dentro do que vimos, independente se utilizamos ou não semicolon, precisamos aderir às regras do ASI. A boa notícia é que muitas delas já empregamos sem nos dar conta, mas como vimos, há pegadinhas.

Porém, **mais prejudicial do que omitir ou não o semicolon é ter no mesmo projeto as duas abordagens**. Não é à toa que guias de estilos começaram a abordar essa questão também. Por exemplo, há o <a href="https://google.github.io/styleguide/jsguide.html" target="_blank">guia de estilo interno da gigante Google</a>, resposável pela criação de aplicações complexas oferecidas na núvem como Gmail, Google Docs, Google Analytics entre outras. Nele, o desenvolvedor é proibido de confiar no ASI, isto é, ele precisa explicitamente adicionar o semicolon. Vejamos o trecho:

>*Every statement must be terminated with a semicolon. Relying on automatic semicolon insertion is forbidden. -- Google JavaScript Style Guide (2002-2017).*

O guia de estilo que vimos é interno, não é uma recomendação universal para que a comunidade a siga. Porém, há dois grandes guias de estilo que invadiram o coletivo imaginário dos desenvolvedores e por este motivo serão destacados a seguir. 

## Style Guides "universais"

Existem vários guias de estilos disponibilizados na web, mas dois merecem destaque devido à popularidade que possuem. De um lado temos o <a href="https://github.com/airbnb/javascript" target="_blank">Airbnb JavaScript Style Guide</a> preconizando o uso do semicolon como estilo a ser seguido. Do outro temos o <a href="https://standardjs.com/" target="blank">JavaScript Standard Style</a> com o objetivo de ser o guia de estilo definitivo da linguagem JavaScript. Nele, o uso do semicolon é desencorajado. Ambos ainda mantêm suas posições na data de publicação deste artigo.

Por mais que tenhamos diferentes guias de estilos com posições distintas a respeito do uso do semicolon, o <a href="https://pt.wikipedia.org/wiki/Zeitgeist" target="_blank">zeitgeist</a> vigente começou a colocar em xeque o *status quo* do semicolon.

## Um novo zeitgeist 

Muitos homens e mulheres começaram a programar em Javascript bem depois da polêmica de 2002. Dessa forma, não internalizaram as amarras do uso onipresente do semicolon, tornando-os mais abertos para experimentarem sua omissão sem remorso. Todavia, esta análise é incompleta se não levarmos em consideração o **fortalecimento das comunidades de frameworks** e a **prática consolidada de builds automatizados** como catalisadores do *semicolonless movement*. 

## A segurança de pertencer a uma comunidade

No mais profundo sentido <a href="https://pt.wikipedia.org/wiki/Zygmunt_Bauman">Baumaniano</a>, nas comunidades que aboliram o semicolon ou que adotaram um style guide que se coadunasse com essa ideia, a omissão por parte de seus membros era protegida e justificada pelo próprio grupo, o que reduzia a ansiedade e a insegurança de seus integrantes. Não havia a necessidade de se discutir tal prática, uma vez que o programador já tinha o beneplácito da própria comunidade. Nesse sentido, a escolha de omitir ou não o semicolon tornou-se mais uma questão de comunidade do que uma decisão pessoal, tirando o peso decisório das costas do programador. Porém, ainda era necessário se munir de aspectos técnicos para que o expurgo do semicolon pudesse ser realizado sem medo. 

## CLI e processos de builds automatizados

Cada comunidade possui um ferramental para lidar com os problemas que se propõe a resolver e com o problema do semicolon não é diferente. Algumas frameworks do mercado utilizam <a href="https://babeljs.io/" target="_blank">Babel</a>, um compilador JavaScript multipropósito, que realiza o ASI durante seu processo de compilação e linters que analisam a integridade do código final. Diferente de 2002, o uso de CLI's (command line interfaces) é condição padrão, não exceção. Hoje temos institucionalizado um processo de build mais rebuscado e menos *error prone* do que no passado.

## Conclusão

A polêmica de usar ou não o semicolon acabou obscurecendo a importância do desenvolvedor conhecer as regras do ASI, regras que possuem impacto independente da abordagem escolhida. 

Comunidades que aboliram o uso desse caracter "tão especial" em prol da **redução de ruído sintático** oferecem todo um ferramental para garantir a integridade do código com mínimo de esforço. Pensar soluções análogas a que vimos em 2002 seria inviável, pois naquela época os CLI, quando existiam, eram embrionários. Todavia, essas medidas visam corrigir, em primeira instância, um código que deveria estar integro se acolhermos a definição de ASI como um procedimento para correção de erro sintático.

Por fim, conclui-se que usar ou não semicolon não é uma decisão puramente do desenvolvedor, mas da comunidade do framework na qual ele participa. É o respaldo da comunidade que dirá se ele esta certo ou errado.


E você? Utiliza ou não semicolon? Deixe sua opinião!