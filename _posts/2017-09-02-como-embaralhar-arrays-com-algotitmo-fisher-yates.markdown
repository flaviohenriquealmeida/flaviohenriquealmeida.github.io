---
layout: post
title:  "Como embaralhar arrays com o algoritmo Fisher–Yates"
description: Apesar do array ser uma estrutura de dados super versátil em JavaScript, não há na especificação uma função que embaralhe seus elementos. Neste post implementaremos o algoritmo Fisher–Yates shuffle para conseguirmos essa funcionalidade.
date:   2017-09-02 17:00:00 -0300
categories:
permalink: /como-embaralhar-arrays-algoritmo-fisher-yates/
author: flavio_almeida
tags: [javascript, shuffle, embaralhar, fisher–yates, algoritmo]
image: logo.png
---

Apesar do array ser uma estrutura de dados super versátil em JavaScript, não há na especificação uma função que embaralhe seus elementos. Neste post implementaremos o algoritmo <a href="https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle" target="_blank">Fisher–Yates shuffle</a> para conseguirmos essa funcionalidade.

## Implementando o algoritimo 

Nosso objetivo será embaralhar uma lista de vogais. Vamos criar a função `embaralha` que recebe uma lista como parâmetro. Iremos iterar em cada elemento da nossa lista começando pelo último elemento até chegar no primeiro:

```javascript
const vogais = ['a', 'e', 'i', 'o', 'u'];

function embaralha(lista) {

    for (let indice = lista.length; indice; indice--) {

    }
}
```
Pode parecer errado definir o valor da variável de incremento `indice = lista.length`, pois sabemos que o índice final de um array é seu tamanho menos um, inclusive utilizar como condição do laço apenas a variável `indice`. Todavia, esses valores atípicos se justificarão ao longo da nossa implementação. Vamos continuar:


```javascript
const vogais = ['a', 'e', 'i', 'o', 'u'];

function embaralha(lista) {

    // indice começa com o tamanho do array
    for (indice = lista.length; indice; indice--) {
        // gerou o índice aleatório
        const indiceAleatorio = Math.floor(Math.random() * indice);
    }
}
```

O resultado de `Math.floor(Math.random() * indice))` sempre resultará em un número entre `0` e `indice - 1`. Faça o teste no console executando várias vezes a instrução `Math.floor(Math.random() * 5))`.

Agora que já temos `indiceAleatorio`, em uma variável auxiliar `elemento` vamos guardar o elemento da lista no índice atual, mas precisaremos utilizar `indice -1`, caso contrário acessaremos uma posição que não existe no array:

```javascript 
const vogais = ['a', 'e', 'i', 'o', 'u'];

function embaralha(lista) {

    for (indice = lista.length; indice; indice--) {

        const indiceAleatorio = Math.floor(Math.random() * indice);
        
        // guarda de um índice aleatório da lista
        const elemento = lista[indice - 1];
    }
}
```

Agora, podemos substituir o elemento em `lista[indice -1]` pelo valor de `lista[indiceAleatorio]`, para em seguida substituirmos o elemento de `lista[indiceAleatorio]` pelo valor que já tínhamos armazenado em `elemento`:

```javascript 
const vogais = ['a', 'e', 'i', 'o', 'u'];

function embaralha(lista) {

    for (let indice = lista.length; indice; indice--) {

        const indiceAleatorio = Math.floor(Math.random() * indice);
        
        // guarda de um índice aleatório da lista
        const elemento = lista[indice - 1];
        
        lista[indice - 1] = lista[indiceAleatorio];
        
        lista[indiceAleatorio] = elemento;
    }
}
```

Já podemos testar o resultado:

```javascript
// código anterior omitido 
embaralha(vogais);
console.log(vogais);
```
 
Por fim, vale um esclarecimento sobre a condição de repetição do laço, no caso `indice`. Qualquer número em JavaScript, seja positivo ou negativo é considerado `true`, exceto o número `0`. Então, a medida que `indice` for decrementado a cada iteração, quando ele chegar a `0`, fará com que o laço termine. Faz todo sentido, porque o quando chegarmos ao primeiro elemento do array não temos mais nenhum outro anterior para trocar de lugar com ele.


Excelente, contudo, podemos simplificar nosso código evitando a declaração da variável `elemento`. Como? É o que veremos a seguir.

## Arrays e atribuição via destructuring

Através de `destructuring` aplicado em arrays podemos reescrever o código da seguinte maneira:

```javascript
const vogais = ['a', 'e', 'i', 'o', 'u'];

function embaralha(lista) {

    for (let indice = lista.length; indice; indice--) {

        const indiceAleatorio = Math.floor(Math.random() * indice);

        // atribuição via destructuring
        [lista[indice - 1], lista[indiceAleatorio]] = 
            [lista[indiceAleatorio], lista[indice - 1]];
    }
}

embaralha(vogais);
console.log(vogais);
```

A sintaxe é um tanto diferente, mas funciona perfeitamente. Nela, estamos indicando através de `[lista[indice - 1], lista[indiceAleatorio]]` que a posição do elemento que estamos iterando e a posição do elemento com índice aleatório receberão, respectivamente os valores de `[lista[indiceAleatorio], lista[indice - 1]`, ou seja, o elemento do índice aleatório e do índice atual.

Podemos ver este recurso em um escopo menor. No exemplo abaixo queremos trocar a posição do segundo elemento pelo primeiro e do primeiro pelo segundo. 

```javascript
const palavras = ['cangaceiro', 'javascript'];
const aux = palavras[0]; // variável auxilia
palavras[0] = palavras[1];
palavras[1] = aux;
console.log(palavras);
```

É claro que para um array de dois elementos um `palavras.reverse()` resolveria o problema facilmente para nós, mas a questão é a possibilidade de reescrevemos o código da seguinte maneira:

```javascript
const palavras = ['cangaceiro', 'javascript'];
[palavras[0], palavras[1]] = [palavras[1], palavras[0]];
console.log(palavras);
```

Perfeito, mas será que podemos enxugar ainda mais nosso código?

## Enxugando ainda mais nosso código

Podemos tranquilamente trocar o verboso `for` por um `while`. Nosso código ficará assim:

```javascript

const vogais = ['a', 'e', 'i', 'o', 'u'];

function embaralha(lista) {

    let indice = lista.length
    
    while(indice) {
        // atenção para o pós-incremento indice-- 
        const indiceAleatorio = Math.floor(Math.random() * indice--);
        [lista[indice], lista[indiceAleatorio]] = 
            [lista[indiceAleatorio], lista[indice]];
    }
}

embaralha(vogais);
console.log(vogais);
```

Bem melhor, não?

## Adicionando no prototype

Podemos ainda adicionar a função `embaralha` no prototype de `Array`, tornando-o uma função disponível por qualquer array em nossa aplicação. Todavia, essa solução é polêmica, pois propriedades adicionadas no prototype podem se chocar com uma propriedade de mesmo nome adicionada mais tarde pela especificação. 

Para ficar mais chique, mudaremos o nome de `embaralha` para `shuffle`:

```javascript
Array.prototype.shuffle = function() {

    let indice = this.length;
    
    while(indice) {

        const indiceAleatorio = Math.floor(Math.random() * indice--);
        [this[indice], this[indiceAleatorio]] = 
            [this[indiceAleatorio], this[indice]];
    }

    return this;
}

const vogais = ['a', 'e', 'i', 'o', 'u'];
vogais.shuffle();
console.log(vogais);
```

## Conclusão

Nem sempre a especificação de uma linguagem nos fornecerá todo arsenal que precisamos para resolver problemas do dia a dia. Tem horas que precisamos levantar as mangas e cair dentro de uma implementação. 