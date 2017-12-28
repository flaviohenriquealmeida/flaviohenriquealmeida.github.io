---
layout: post
title:  "Emulando parâmetros nomeados em JavaScript através de destructuring assignment"
description: Neste artigo aprenderemos como emular em Javascript **parâmetros nomeados** através da atribuição via desestruturação (destructuring assignment). 
date:   2017-12-28 08:00:00 -0300
categories:
permalink: /emulando-parametros-nomeados-javascript-atraves-destructuring-assignment/
author: flavio_almeida
tags: [named parameters, destructuring assignment, javascript, ES2015, ES6]
image: logo.png
---

Neste artigo aprenderemos como emular em Javascript **parâmetros nomeados** através da atribuição via desestruturação (destructuring assignment). 

## O problema

Não é raro desenvolvedores procurarem entender um código lendo-o diretamente em vez de consultar sua documentação. **O código nunca mente, ou ele funciona ou não funciona**, já uma documentação pode ser incompleta ou até mesmo desatualizada. Nesse sentido, facilitar a leitura é importante não só para terceiros, mas também para quem desenvolve o código.

Vejamos o trecho de um código que chama a função `moveFrame`:

```javascript
// código anterior omitido 

moveFrame ('sprite1', 'sprite2');

// código posterior omitido 
```

Para quem lê o código não fica claro o papel de cada parâmetro passado para a função, sendo necessário recorrer à sua declaração. Vejamos:

```javascript 
const moveFrame = (from, to) => {
    /*
        Acessa o elementod DOM
        removendo e adicionando classes
    */
    console.log(from);
    console.log(to);
};
```

Agora sabemos que o primeiro parâmetro é o *frame de origem* e o segundo o *frame final*. 

Uma maneira de tentarmos deixar mais claro a natureza desses dois parâmetros é alterar o nome da função, por exemplo, para `moveFrameFromTo`. Com essa mudança, o programador que se deparar com a chamada da função, com um pouquinho de esforço, pode inferir o papel de cada parâmetro passado para a função. 

Todavia, tal artifício não seria necessário se o JavaScript suportasse parâmetros nomeados como na linguagem Python. 

Vejamos o mesmo exemplo na linguagem Python:

```python
def move_frame (from, to):
    # faz alguma coisa com os parâmetros
```

```python
# chamando a função
move_frame (from="sprite1", to="sprite2")
```

Para quem lê a chamada da função, fica evidente que "sprite1" se refere ao parâmetro `from` e "sprite2" ao parâmetro `to`. Isso evitará que ela tenha que consultar a definição da função para saber o contexto de cada parâmetro. Ainda há mais uma vantagem nessa abordagem.

Se a ordem dos parâmetros nomeados da função for invertida, ela continuará com o comportamento esperado. Em suma, não precisamos nos preocupar com a ordem dos parâmetros:

```python
# chamando a função com a posição dos parâmetros invertida
move_frame (to="sprite2", from="sprite1")
```

>Mesmo com parâmetros nomeados, o desenvolvedor que nunca trabalhou com a função ou não lembra dos seus parâmetros precisará consultá-la para verificar os parâmetros suportados. No entanto, isso não invalida a melhora na legibilidade do código e a vantagem de não precisarmos passar os parâmetros em uma ordem específica.

Será que conseguimos aplicar essa mesma estratégia em nosso código Javascript, mesmo ela não suportando parâmetros nomeados? É o que veremos a seguir. 

## Primeira tentativa. Recebendo um objeto como parâmetro

Vamos alterar primeiro a chamada da função `moveFrame`. Ela receberá um objeto JavaScript cuja as propriedades são os parâmetros que precisa receber:

```javascript
moveFrame ({ from: 'sprite1', to: 'sprite2' });
```

Excelente, mas do jeito que esta, nossa função `moveFrame` precisará receber um objeto JavaScript. Um desvantagem dessa abordagem é não sabermos identificar rapidamente quais são as propriedades do objeto recebido que serão consideradas "parâmetros" da função. Por fim, se estivermos modificando um código já existente seremos obrigados a mudar o acesso às variáveis `from` e `to` para `parameters.from` e `parameters.to` respectivamente:

```javascript
/*
    Ok, recebemos "parameters", um objeto. Mas quais 
    são as chaves que usaremos como parâmetro?
*/
const moveFrame = parameters => {

   /* 
    Seremos obrigados a acessar 
    as proprieades do objeto
   */
   console.log(parameters.from);
   console.log(parameters.to);
};
```

Mas nem tudo esta perdido. A boa notícia é que podemos conseguir um resultado parecido com o que vimos na linguagem Python emulando parâmetros nomeados através de *destructuring assignment*.

## Emulando parâmetros nomeados através de destructuring assignment

Vamos alterar o parâmetro de `moveFrame`. Ele ficará assim:

```javascript
const moveFrame = ({ from, to }) => {
   /*
    Mudamos o parâmetro, mas o restante do código 
    continou como estava
   */
   console.log(from);
   console.log(to);
};
```

A nossa chamada passando um objeto como parâmetro continua funcionando:

```javascript
moveFrame ({ from: 'sprite1', to: 'sprite2' });
```

Perfeito! Utilizamos as chaves de um objeto para nomear os parâmetros recebidos pela função e através de destructuring assignment tratamos cada propriedade como variáveis individuais dentro da função. Podemos até inverter a ordem das propriedades que isso não impactará em nosso código:

```javascript
moveFrame ({ to: 'sprite2', from: 'sprite1' });
```

Para que o leitor perceba com mais clareza o que esta contecendo por debaixo dos panos, segue um exemplo de escopo menor:

```javascript
const parameters = {
    from: 'sprite1',
    to: 'sprite2', 
};
// atribuição via desestruturaçãp
const { from, to } = parameters;
// imprime "sprite1"
console.log(from);
// imprime "sprite2"
console.log(to);
```

## Conclusão

Tomar como inspiração o que há de melhor em outras linguagens e tentar aplicá-la em sua linguagem é colocar em prática a sabedoria do programador. 

E você? Já utilizava essa estratégia antes? Há recursos de outras linguagens que você gostaria que existisse em JavaScript? Deixe sua opinião.