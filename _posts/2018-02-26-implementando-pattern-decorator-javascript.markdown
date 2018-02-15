---
layout: post
title:  "Implementando o pattern Decorator em JavaScript"
description: Neste artigo aplicaremos o padrão de projeto Decorator dentro do contexto dinâmico da linguagem JavaScript. Estruturaremos nossa solução de uma maneira padronizada, favorecendo assim a melhor organização do código e adoção por outros desenvolvedores.
date: 2018-02-15 08:00:00 -0300
categories:
permalink: /implementando-pattern-decorator-javascript/
author: flavio_almeida
tags: [javascript, decorator, design pattern]
image: logo.png
---
Neste artigo aplicaremos o padrão de projeto Decorator dentro do contexto dinâmico da linguagem JavaScript. Estruturaremos nossa solução de uma maneira padronizada, favorecendo assim a melhor organização do código e adoção por outros desenvolvedores.

## O problema

Temos a seguinte classe `Person`:

```javascript
// app/models/person.js;
export class Person {

    constructor(name, surname) {
        this._name = name;
        this._surname = surname;
    }

    speak(phrase) {
        console.log(phrase);
    }

    getFullName() {
        return `${this._name} ${this._surname}`;
    }
}
```

Vejamos a classe em ação:

```javascript
// app/app.js
import { Person } from './models/person.js';

const person1 = new Person('Flávio', 'Almeida');
const person2 = new Person('Almeida', 'Flávio');

person1.speak('Cangaceiro Javascript');
console.log(person1.getFullName());
// Flávio Almeida

person2.speak('Calopsita');
console.log(person2.getFullName());
// Almeida Flávio
```

Excelente, mas se quisemos que o método `getFullName()` de `person2` separe o `name`  do `surname` através de um hífen? Não podemos mudar diretamente na classe `Person` pois isso afetará a instância `person1`. Uma solução é modificamos o método `Person.getFullName` fazendo-o receber como parâmetro o separador. Todavia, essa abordagem ferirá a máxima do *open/close principle*. Sua definição na <a href="https://en.wikipedia.org/wiki/Open/closed_principle" target="_blank">wikiipedia</a>:

>"software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification"; that is, such an entity can allow its behaviour to be extended without modifying its source code."

Uma possível solução é usar herança, isto é, criar uma nova classe que estenda `Person` e que fará a sobrescrita do método `getFullName()`. 

>*Vale lembrar a sintaxe `class` e a instrução `extends` são um atalho, isto é, açúcar sintático para criação de funções construtoras (obrigando a utilizar o operador new) e a adição de métodos ao prototype respectivamente.*

No entanto, se quisermos adicionar novos comportamentos teremos uma explosão no número de classes. É aí que entra o pattern **Decorator**.

## O padrão de projeto Decorator

Segundo o livro *Design Patterns: Elements of Reusable Object-Oriented Software* o Decorator "is a design pattern that allows behavior to be added to an individual object, either statically or dynamically, without affecting the behavior of other objects from the same class.". 

Em essência, a beleza do deste pattern está na preservação da classe original, pois apenas aquelas instâncias da classe escolhidas pelo desenvolvedor serão modificadas. Vamos aplicar esse pattern ao nosso problema.

## Primeira solução

Pela característica dinâmica da linguagem JavaScript, podemos simplesmente substituir o método original por um novo que chamará o método original permitindo assim modificar seu comportamento original.

```javascript
// app/app.js
import { Person } from './models/person.js';

const person1 = new Person('Flávio', 'Almeida');
const person2 = new Person('Almeida', 'Flávio');

person1.speak('Cangaceiro Javascript');
console.log(person1.getFullName());
// Flávio Almeida

// guarda o método original
const originalMethod = person2.getFullName.bind(person2);
// adicionando um novo método que chama o original
person2.getFullName = () => {
    return originalMethod().replace(/ /, '-');
};

console.log(person2.getFullName());
// Almeida-Flávio
```

 Excelente, mas se tivéssemos 10 instâncias de `Person` que necessitassem esse novo comportamento? Em pouco tempo nosso código se tornaria um código difícil de manter.

## Segunda solução

Queremos uma solução padronizada e mais fácil de manter. Porém, antes de implementá-la, vejamos como fica o módulo `app/app.js` utilizando essa solução:

```javascript
// app/app.js
import { Person } from './models/person.js';
// importou um decorator, isolado em seu próprio módulo
import { getFullNameWithHyphen } from './models/decorators.js';
// importou a função utilitária decorate
import { decorate } from './utils/decorate.js';

const person1 = new Person('Flávio', 'Almeida');
const person2 = new Person('Almeida', 'Flávio');

person1.speak('Cangaceiro Javascript');
console.log(person1.getFullName());
// Flávio Almeida

// decorou o método gerFullName com getFullNameWithHyphen
decorate(person2, {
    getFullName: getFullNameWithHyphen
});

console.log(person2.getFullName());
// Almeida-Flávio
```

Temos a função utilitária `decorate` que receberá dois parâmetros. O primeiro é a *instância* do objeto que desejamos decorar. Já o segundo é o objeto *handler*. As propriedades do objeto *handler* equivalem aos nomes dos métodos que desejamos decorar. Seu valor será o decorator que desejamos aplicar.

Vamos dar uma olhada na implementação do decorator `getFullNameWithHyphen`:

```javascript
// app/models/decorators.js 
export const getFullNameWithHyphen = (original, propertyName, args) => {
    console.log(`Propriedade decorada ${propertyName}`);
    console.log(`Argumentos do método ${args}`);
    const result = original(...args).replace(/ /, '-');
    console.log(`resultado modificado: ${result}`)
    return result;
}
```
É a função utilitária `decorate` que passará os parâmetros que nosso decorator precisa. Em ordem, esses parâmetros são:

* O método original a ser decorado
* O nome do método
* Os parâmetros que o método recebe ou não

Sabendo o que cada parâmetro representa, fica fácil entender a lógica do nosso decorator. Executamos o método original modificando seu resultado e é este valor que retornamos. O restante do código são as mensagens no console para que possamos ver o que esta acontecendo. 

A grande questão agora é implementarmos a função `decorate`, centro nervoso da nossa solução.

## Implementando a função decorate

Vamos criar o módulo `app/utils/decorate.js`. Nele, vamos declarar a função `decorate` que receberá o `target` (a instância alvo) e o `handler`:

```javascript
// app/utils/decorate.js
export const decorate = (target, handler) => // falta o resto
```

A primeira coisa que faremos é listar todas as chaves do objeto `handler`, pois são elas que indicam qual método decorar da instância alvo. Todavia, só podemos iterar nas propriedades do próprio objeto e não do seu prototype, no caso `Object`. Se iteramos nas propriedades do `prototype` acabaremos encontrando outras chaves que não dizem respeito aos métodos que desejamos decorar. A função `Object.keys()` nos atende muito bem:

```javascript
// app/utils/decorate.js
export const decorate = (target, handler) => 
    Object.keys(handler).forEach(propertyName => {
        
            
    });
```        
Agora, em cada iteração, precisamos guardar o método original, um dos parâmetros que os nossos decorators dependem:

```javascript
// app/utils/decorate.js
export const decorate = (target, handler) => 
    Object.keys(handler).forEach(propertyName => {
        // guarda a função original
        const original = target[propertyName].bind(target);
    });
```

Através de `target[propertyName]` temos acesso ao método que será decorado. Não podemos nos esquecer de realizar o `bind` da nova referência do método com seu contexto inicial, `target`. Se tivéssemos omitido esse passo, o `this` de `original` não apontaria mais para a instância de `target`.

Agora precisamos decorar o método da instância de `target`:

```javascript
// app/utils/decorate.js
export const decorate = (target, handler) => 
    Object.keys(handler).forEach(propertyName => {
        // guarda a função original
        const original = target[propertyName].bind(target);

        target[propertyName] = (...args) => 
            handler[propertyName](original, propertyName, args);
    });
```

Como não sabemos quantos parâmetros o método original recebe, usamos o *REST operator* na função que substituirá o método original em `target`. No bloco da função, invocamos o decorator passado para o `handler` através de `handler[propertyName]()`, por isso é importante que o nome das propriedades do handler tenham o mesmo nome do método que será decorado. Por fim, passamos para `handler[propertyName]()` os valores de `original`, `propertyName`, e `args`. 

Se olharmos mais uma vez nosso decorator `getFullNameWithHyphen` constatamos que os parâmetros são recebidos corretamente:

```javascript
// app/models/decorators.js
export const getFullNameWithHyphen = (original, propertyName, args) => {
    console.log(`Propriedade decorada ${propertyName}`);
    console.log(`Argumentos do método ${args}`);
    const result = original(...args).replace(/ /, '-');
    console.log(`resultado modificado: ${result}`)
    return result;
}
```

Por fim, nosso `app/app.js` ficará assim:

```javascript
// app/app.js 
import { Person } from './models/person.js';
import { getFullNameWithHyphen } from './models/decorators.js';
import { decorate } from './utils/decorate.js';

const person1 = new Person('Flávio', 'Almeida');
const person2 = new Person('Almeida', 'Flávio');

person1.speak('Cangaceiro Javascript');
console.log(person1.getFullName());

decorate(person2, {
    getFullName: getFullNameWithHyphen
});

console.log(person2.getFullName());
// Almeida-Flávio
```

Você encontra o código completo deste arquivo no meu <a href="https://github.com/flaviohenriquealmeida/decorator-pattern-javascript-implementation" target="_blank">github</a>. 

## Conclusão

Apesar do suporte a decorator ter sido especificado na linguagem JavaScript, ainda não há um consenso de sua implementação. Alguns frameworks do mercado, inclusive a linguagem TypeScript suportam este recurso com sintaxe `@`, a mesma utilizada na linguagem Python que há anos suporta nativamente este recurso. Todavia, nada nos impede de implementá-lo em vanilla JavaScript. E você? Já precisou estender um objeto um objeto sem modificar sua classe? Deixe sua opinião.

