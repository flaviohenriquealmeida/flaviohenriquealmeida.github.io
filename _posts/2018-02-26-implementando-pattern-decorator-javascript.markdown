---
layout: post
title:  "Suportando Decorators com vanilla JavaScript"
description: Não há suporte nativo a decorators na linguagem JavaScript (ano base 2018), apesar de existir formalmente uma proposta. Neste artigo Implementaremos uma solução padronizada para que possamos utilizar decorators com vanilla JavaScript hoje.
date: 2018-02-26 08:00:00 -0300
categories:
permalink: /suportando-decorators-com-vanilla-javascript/
author: flavio_almeida
tags: [javascript, decorator, design pattern, decorate]
image: logo.png
---
Não há suporte nativo a *decorators* na linguagem JavaScript (ano base 2018), apesar de existir formalmente uma <a href="https://github.com/tc39/proposal-decorators" target="_blank">proposta</a> em andamento. Neste artigo Implementaremos uma solução padronizada para que possamos utilizar decorators com vanilla JavaScript hoje.

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
        return `${this._name} is speaking... ${phrase}`
    }

    getFullName() {
        return `${this._name} ${this._surname}`;
    }
}
```

Queremos logar o tempo de execução dos métodos `speak` e `getFullName`. Uma solução é alterarmos os métodos adicionando todo o código necessário para cronometrar o tempo:

```javascript
// app/models/person.js;
export class Person {

    constructor(name, surname) {
        this._name = name;
        this._surname = surname;
    }

    speak(phrase) {
        console.time('speak');
        const result = `${this._name} is speaking... ${phrase}`;
        console.timeEnd('speak');
        return result;
    }

    getFullName() {
        console.time('speak');
        const result = `${this._name} ${this._surname}`;
        console.timeEnd('speak');
        return result;
    }
}
```

Não precisamos meditar muito para verificarmos que duplicamos nosso código. Aliás, teríamos ainda mais código duplicado se outras classes da nossa aplicação precisassem cronometrar também a execução de seus métodos. 

Uma solução é isolar o código duplicado em um único lugar e aplicá-lo aos métodos da classe. Aliás, outras linguagem já resolveram essa questão adicionando em sua sintaxe o suporte a **decorators**. 

## Decorators

Decorators são suportados nativamente em linguagens como Python, Java, C# e por outras linguagens, inclusive TypeScript. Em suma, um decorator nada mais é do que um trecho de código isolado aplicável em uma ou mais funções, inclusive em métodos de classe. A lógica a ser aplicada depende do problema a ser resolvido, mas o mais importante é que teremos essa lógica em um lugar apenas, evitando assim duplicação de código.

Vejamos a seguir um exemplo do uso de decorators na linguagem TypeScript.

## TypeScript e decorators

Em TypeScript, podemos isolar um código e aplicá-lo em métodos de classes através da sintaxe `@nomeDoDecorator`:

```javascript
// app/models/person.js;
export class Person {

    constructor(name, surname) {
        this._name = name;
        this._surname = surname;
    }

    @logExecutionTime
    speak(phrase) {
        return `${this._name} is speaking... ${phrase}`
    }

    @logExecutionTime
    getFullName() {
        return `${this._name} ${this._surname}`;
    }
}
```

>*A implementação da função `logExecutionTime` em TypeScript foi omitida, pois focaremos na implementação da nossa solução.*

Será que podemos chegar a um resultado semelhante diretamente na JavaScript, inclusive sem o uso de um transcompilador como Babel? Vamos tentar.

## Primeira solução

Pela característica dinâmica da linguagem JavaScript, podemos alterar diretamente no prototype de `Person` o método `speak`. Para isso, precisamos guardar o método original para então chamá-lo na nova função que o substituirá:

```javascript
// app/app.js
import { Person } from './models/person.js';

// guardou o método original
const method = Person.prototype.speak;

// substitui o método por uma função
Person.prototype.speak = function (...args) {
    console.log(`Argumentos do método: ${args}`);
    console.time('speak');
    // executa o código original
    const result = method.bind(this)(...args);
    console.log(`Resultado do método: ${result}`);
    console.timeEnd('speak');
    return result;
};

const person = new Person('Flávio', 'Almeida');
person.speak('Canganceiro JavaScript');
```
>*Foi utilizada a sintaxe de carregamento nativo de módulos suportado pelo Google Chrome. Saiba mais no artigo <a href="http://cangaceirojavascript.com.br/importacao-nativa-modulos-browser/" target="_blank">"Importação nativa de módulos no browser"</a>, deste mesmo autor.*

Vamos analisar o código. Guardarmos uma referência para o método definido no prototype da classe `Person` na variável `method`. 

>*Não esqueça que ao declararmos uma classe com a sintaxe `class` as definições de seus métodos são adicionadas em seu prototype. É por isso que acessarmos `Person.prototype.speak` e não `Person.speak`.*

Com o método original guardado, substituímos `Person.prototype.speak` por uma por função que redefiná o método. Ela recebe um número indeterminado de parâmetros através do *Rest operator*, pois não sabemos quantos parâmetros o método original recebe. 

Vale ressaltar que  utilizamos uma função no lugar de uma *arrow function* pois necessitamos de um escopo dinâmico, isto é, precisamos que o `this` seja a instância que invoca o método naquele momento. 

Dentro da função que define o novo método, executamos um código arbitrário antes e depois da chamada do método original, todavia, um trecho de código merece destaque:

```javascript
const result = method.bind(this)(...args);
```
Através de `method.bind(this)` criamos uma nova referência para o método original que tem como contexto o `this` da nova função. Lembre-se que esse `this` referenciará a instância que estiver invocando o novo método. Em seguida, passamos através do *spread operator* cada parâmetro recebido no array `args` como parâmetros individuais para a função. 

Executando nosso código, a saída do console será:

```bash
Método chamado com os parâmetros: Canganceiro JavaScript
Resultado do método: Flávio is speaking... Canganceiro JavaScript
speak: 0.494873046875ms
```

Excelente, mas essa estratégia deixa a desejar. Lembrem-se que precisamos aplicar a mesma lógica ao método `getFullName` e em outras classes quando necessário. Sendo assim, precisamos padronizar nossa solução em algo que seja fácil de usar pelos demais desenvolvedores da equipe.

## Isolando decorators e definindo uma API

Para facilitar a vida do desenvolvedor, queremos a seguinte solução:

```javascript
// app/app.js
import { Person } from './models/person.js';
// as demais funções ainda não existem!
import { decorate } from './utils/decorate.js';
import { logExecutionTime } from './models/decorators.js';

decorate(Person, {
    speak: logExecutionTime, 
    getFullName: logExecutionTime
});

const person = new Person('Flávio', 'Almeida');
person.speak('Canganceiro JavaScript');
```

Na solução que acabamos de ver temos a função utilitária `decorate` receberá dois parâmetros. O primeiro é a *classe* que desejamos decorar seus métodos. O segundo é o objeto *handler*. **As propriedades do objeto *handler* equivalem aos nomes dos métodos que desejamos decorar na classe**. Seu valor será o decorator que desejamos aplicar.

Vamos dar uma olhada na implementação do decorator `logExecutionTime`:

```javascript
// app/models/decorators.js 
export const logExecutionTime = (method, property, args) => {
    console.log(`Método decorado: ${property}`);
    console.log(`Argumentos do método ${args}`);
    console.time(property);
    const result = method(...args);
    console.timeEnd(property);
    console.log(`resultado do método: ${result}`)
    return result;
};
```
É a função utilitária `decorate` que passará os parâmetros que nosso decorator precisa. Em ordem, esses parâmetros são:

* O método original a ser decorado, com seu contexto já modificado
* O nome do método
* Os parâmetros que o método recebe (ou não)

O mais interessante é que **definimos uma API para criação de decorators**. Essa padronização é importante, pois permitirá que a solução seja utilizada por outros desenvolvedores mais facilmente. Sabendo o que cada parâmetro representa, fica fácil entender a lógica do nosso decorator.

A grande questão agora é a implementação da função utilitária `decorate`, centro nervoso da nossa solução.

## Implementando a função decorate

Vamos criar o módulo `app/utils/decorate.js`. Nele declararemos a função `decorate` que receberá a `clazz` e o `handler`:

```javascript
// app/utils/decorate.js
export const decorate = (clazz, handler) => // falta o resto
```

A primeira coisa que faremos é listar todas as chaves do objeto `handler`, pois são elas que indicam qual método decorar da classe. 


Todavia, só podemos iterar nas propriedades do próprio objeto `handler` (aquelas que adicionamos ao criá-lo) e não do seu prototype, no caso `Object`. Se iteramos nas propriedades do `prototype` acabaremos encontrando outras chaves que não dizem respeito aos métodos que desejamos decorar. A função `Object.keys()` nos atende muito bem:

```javascript
// app/utils/decorate.js
export const decorate = (target, handler) => 
    // retorna todas as propriedades enumeráveis do objeto
    Object.keys(handler).forEach(property => {
        // falta implementar     
    });
```        

Em cada iteração precisaremos guardar o método original, um dos parâmetros que nossos decorators dependem:

```javascript
// app/utils/decorate.js
export const decorate = (target, handler) => 
    Object.keys(handler).forEach(property => {
        const method = clazz.prototype[property];
    });
```

Armazenamos na variável `method` uma referência para o método que será decorado. Agora precisamos fazer a mesma coisa, mas dessa vez para armazenar o decorator do método:

```javascript
// app/utils/decorate.js
export const decorate = (target, handler) => 
    Object.keys(handler).forEach(property => {
        const method = clazz.prototype[property];
        const decorator = handler[property];
    });
```

Ótimo! Só precisamos modificar o método original da classe para que invoque nosso decorator. Lembrando que o decorator receberá como parâmetro uma referência para o método original, inclusive já associado ao `this` da função que substituiu o método na classe:


```javascript
// app/utils/decorate.js
export const decorate = (clazz, handler) => {
    Object.keys(handler).forEach(property => {
        const method = clazz.prototype[property];
        const decorator = handler[property];
        // Adiciona novo método que ao ser chamado 
        // chamará o decorator por debaixo dos panos
        clazz.prototype[property] = function (...args) {
            return decorator(method.bind(this), property, args);
        };  
    });    
};
```
Excelente, o código que escrevemos até agora é suficiente para que nosso decorator seja aplicado. Mas se quisermos aplicar mais de um decorator por método? 

## Métodos com mais de um decorator

O decorator `logExecutionTime` esta com muita responsabilidade. Extrairemos dele o código que loga os dados da função como o seu nome, parâmetros recebidos e seu retorno em um novo decorator que chamaremos de `inspectMethod`:

Alterando `app/models/decorators.js`:

```javascript
// app/models/decorators.js
export const logExecutionTime = (method, property, args) => {
    console.time(property);
    const result = method(...args);
    console.timeEnd(property);
    return result;
};

export const inspectMethod = (method, property, args) => {
    console.log(`Método decorado: ${property}`);
    console.log(`Argumentos do método ${args}`);
    const result = method(...args);
    console.log(`resultado do método: ${result}`)
    return result;
};
```
>*Reparem que nossos decorators seguem a mesma API, fantástico!*

Agora, alterando `app/app.js` para fazer uso do nosso novo decorator:

```javascript
import { Person } from './models/person.js';
import { decorate } from './utils/decorate.js';
import { logExecutionTime, inspectMethod } from './models/decorators.js';

// passando um array de decorators
decorate(Person, {
    speak: [inspectMethod, logExecutionTime],
    getFullName: [logExecutionTime]
});

const person = new Person('Flávio', 'Almeida');
person.speak('Canganceiro JavaScript');
```

Recebemos um erro no console:

```javascript
Uncaught TypeError: decorator is not a function
    at Person.clazz.(anonymous function)
```

Infelizmente a função `decorate` não esta preparada para lidar com um `Array` de decorators. Precisamos alterá-la para que funcione:

```javascript
// app/utils/decorate.js
export const decorate = (clazz, handler) => {
    Object.keys(handler).forEach(property => {
        const decorators = handler[property];
        decorators.forEach(decorator => {
            // o método já pode ter sido decorado antes
            const method = clazz.prototype[property];
            clazz.prototype[property] = function (...args) {
                return decorator(method.bind(this), property, args);
            };  
        });
    });    
};
```

Excelente, agora podemos combinar decorators em um mesmo método.

## Ordem dos decorators

Se analisarmos atentamente a aplicação dos decorators, veremos que são aplicados da direita para a esquerda. Para facilitar o entendimento, vamos alterar nosso código para que o primeiro decorator da lista seja o primeiro a ser aplicado e assim por diante.

```javascript
// app/util/decorate.js
export const decorate = (clazz, handler) => {

    Object.keys(handler).forEach(property => {
        // faz reverse
        const decorators = handler[property].reverse();
        decorators.forEach(decorator => {
            const method = clazz.prototype[property];
            clazz.prototype[property] = function (...args) {
                return decorator(method.bind(this), property, args);
            };  
        });
    });    
};
```

Agora, vamos alterar a ordem dos decorators em `app/app.js`:

```javascript
// app/app.js
// código anterior omitido 
// agora aplicará da esquerda para a direita
decorate(Person, {
    speak: [logExecutionTime, inspectMethod],
    getFullName: [logExecutionTime]
});
// código posterior omitido
```

Agora fica mais fácil para quem lê o código entender a ordem de aplicação dos decorators. 

Tudo excelente, mas se nosso decorator precisar receber uma configuração incial?

## Decorators que recebem parâmetros

Criamos o decorator `inspectMethod`, mas nem sempre queremos logar o resultado do método, apenas seus parâmetros. Podemos atender a este requisito facilmente. Primeiro, vamos alterar o decorator `inspectMethod`:

```javascript
export const inspectMethod = ({ excludeReturn } = {}) => 
    (method, property, args) => {
        console.log(`Método decorado: ${property}`);
        console.log(`Argumentos do método ${args}`);
        const result = method(...args);
        if(!excludeReturn) console.log(`resultado do método: ${result}`)
        return result;
    };
```
Vejam que nosso decorator recebe como parâmetro um objeto JavaScript que ao sofrer o *destructuring* disponibilizará a variável `excludeReturn`, utilizada para controlar a exibição da variável `result`. Por fim,  o retorno do decorator será a função que adere à API que define nossos decorators. 

Alterando o módulo `app/app.js`:

```javascript
import { Person } from './models/person.js';
import { decorate } from './utils/decorate.js';
import { logExecutionTime, inspectMethod } from './models/decorators.js';

decorate(Person, {
    speak: [inspectMethod({ excludeReturn: true }), logExecutionTime],
    getFullName: [logExecutionTime]
});

const person = new Person('Flávio', 'Almeida');
person.speak('Canganceiro JavaScript');
person.getFullName();
```
Se não passarmos parâmetro nenhum para `inspectMethod` o padrão continuará sendo inspecionar o retorno do método. Agora podemos escolher logar ou não o retorno dos métodos.

Podemos tornar ainda melhor a maneira pela qual aplicamos nossos decorators.

## Aplicando o decorator diretamente na definição da classe:

É possível decorar a classe diretamente no módulo no qual foi declarada. Vamos alterar `app/models/person.js`:

```javascript
import { decorate } from '../utils/decorate.js';
import { logExecutionTime, inspectMethod } from './decorators.js';

export class Person {

    constructor(name, surname) {
        this._name = name;
        this._surname = surname;
    }

    speak(phrase) {
        return `${this._name} is speaking... ${phrase}`
    }

    getFullName() {
        return `${this._name} ${this._surname}`;
    }
}

decorate(Person, {
    speak: [inspectMethod({ excludeReturn: true }), logExecutionTime],
    getFullName: [logExecutionTime]
});
```

Por fim, nosso nódulo `app/app.js` ficará dessa forma:

```javascript
// app/app.js
import { Person } from './models/person.js';

const person = new Person('Flávio', 'Almeida');
person.speak('Canganceiro JavaScript');
person.getFullName();
```

Nosso módulo dependerá dos decorators e da função `decorate`. Se o JavaScript suportasse nativamente decorators, dependeríamos apenas dos decorators, além de podermos aplicá-los com a sintaxe `@`, aliás, muito sexy.

## Código no Github

Você encontra o código completo deste artigo no meu <a href="https://github.com/flaviohenriquealmeida/decorator-pattern-javascript-implementation" target="_blank">github</a>. 

## Conclusão

Apesar do suporte a decorator ter sido especificado na linguagem JavaScript, ainda não há um consenso sobre sua implementação. Alguns frameworks do mercado, inclusive a linguagem TypeScript suportam este recurso com sintaxe `@`, a mesma utilizada na linguagem Python que há anos suporta nativamente este recurso. Todavia, nada nos impede de implementá-lo em vanilla JavaScript. E você? Deixe sua opnião!