---
layout: post
title:  "Class Factory ou Factory Function?"
description: .
date:   2017-09-25 14:00:00 -0300
categories:
permalink: /padrao-de-projeto-factory-ou-factory-function/
author: flavio_almeida
tags: [javascript, class factory, factory function]
image: logo.png
---

JavaScript é uma linguagem multi paradigma suportando a programação orientada a objetos e a funcional. Contudo, *grandes poderes trazem grandes responsabilidades* e muitas vezes ficamos na dúvida qual deles aplicar para resolver determinado problema. 

Neste post teremos um problema e utilizaremos diferentes paradigmas para resolvê-lo. Vamos ao problema.

## O problema

Precisamos isolar a complexidade de criação de proxies em nossa aplicação para facilitar sua construção em diversas partes do sistema. 

Dentro do *paradigma orientado a objetos*, há o padrão de projeto **Factory** para lidar com problemas como o citado. Já no *paradigma funcional* podemos trabalhar com **Factory Functions**. Veremos a solução com os dois paradigmas para que o leitor possa avaliar cada um deles. Iniciaremos pelo orientado a objetos.

## Class Factory

Vejamos um exemplo utilizado o paradigma da orientação a objetos com a classe `ProxyFactory`. Seu método estático `createProxy` recebe como parâmetro um objeto e retorna um proxy deste mesmo objeto que ao ter suas propriedade lidas exibe no console o nome da propriedade acessada:

```javascript
// paradigma orientado à objetos
class ProxyFactory {
    
    createProxy(object) {
        return new Proxy(object, {
            get(target, property, receiver) {
                console.log(`A propriedade "${property}" foi lida.`);
                return target[property];
            }
        });
    }
}

let person = { name: 'Flávio Almeida' };
// cria um instância do ProxyFactory
const factory = new ProxyFactory();
// cria o proxy
person = factory.createProxy(person);
console.log(person.name); // exibe também a mensagem do proxy
```

No entanto, como a classe `ProxyFactory` não possui estado interno, podemos converter seu método `createProxy` para um método estático permitindo acessá-lo diretamente da classe sem a necessidade de uma instância:

```javascript
// paradigma orientado à objetos
class ProxyFactory {

    static createProxy(object) {
        return new Proxy(object, {
            get(target, property, receiver) {
                console.log(`A propriedade "${property}" foi lida.`);
                return target[property];
            }
        });
    }
}

let person = { name: 'Flávio Almeida' };
// não precisou criar a instância
person = ProxyFactory.createProxy(person);
console.log(person.name); // exibe também a mensagem do proxy
```

Com essa modificação, não faz sentido o programador criar instâncias de `ProxyFactory`, pois o método `createProxy` só pode ser acessado diretamente pela classe. Para garantirmos isso, podemos lançar uma exeção caso o operador `new` seja utilizado:

```javascript
// paradigma orientado à objetos
class ProxyFactory {
    // blidando o uso do operador new
    constructor() {
        throw new Error('Não é permitida a criação de instâncias desta classe');
    }

    static createProxy(object) {
        return new Proxy(object, {
            get(target, property, receiver) {
                console.log(`A propriedade "${property}" foi lida.`);
                return target[property];
            }
        });
    }
}

// vai resultar em um erro em tempo de execução
const factory = new ProxyFactory();
```

Por outro lado, podemos rescrever nossa classe através do paradigma funcional. 

## Factory Function

Vamos declarar apenas a função `createProxy`. Diferente de outras linguagens como Java ou C#, funções podem existir sem pertencerem a uma classe:

```javascript
// paradigma funcional
const createProxy = object => new Proxy(object, {
    get(target, property, receiver) {
        console.log(`A propriedade "${property}" foi lida.`);
        return target[property];
    }
});

let person = { name: 'Flávio Almeida' };
person = createProxy(person);
console.log(person.name);
```

Fica evidente que temos um código mais limpo. Primeiro, declaramos `createProxy` como uma `function expression` com auxílio de uma `arrow function` sem bloco e que assume automaticamente como retorno o proxy criado. Não gastamos linhas de código declarando uma classe e por conseguinte não foi necessário blindar seu `constructor` para evitar a criação de instâncias.

## Conclusão

No contexto do problema que vimos, utilizar uma `factory function` mostrou-se uma solução menos verbosa e fácil de manter do que a sua versão baseada em classe. Contudo, a aplicação de um paradigma depende muito do problema a ser resolvido, além da bagagem do próprio programador. 

E você? Consegue dar um exemplo no qual o paradigma da orientação a objetos seja mais elegante que o funcional? 