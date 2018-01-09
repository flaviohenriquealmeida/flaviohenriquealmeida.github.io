---
layout: post
title:  "Os limites da recursão em JavaScript, TCO e o padrão Trampoline"
description: XXX
date: 2018-01-08 06:00:00 -0300
categories:
permalink: /limites-recursao-javascript-tco-e-padrao-trampoline/
author: flavio_almeida
tags: [javascript, recursion, trampoline, functional programing]
image: logo.png
---
JavaScript é uma linguagem multiparadigma suportando os paradigmas da orientação a objetos e o funcional. Não é raro desenvolvedores mais voltados para o paradigma funcional se valerem de **recursão** para solucionar problemas. Porém, pela ausência de *tail call optimization* (**TCO**) nas engines do mercado, longas chamadas recursivas podem estourar a *call stack* terminando abruptamente o programa. Neste artigo aprenderemos a utilizar recursão sem medo mesmo com ausência de TCO através do *pattern* **Trampoline**.

Primeiro, veremos um pouco sobre recursão e os problemas que podem acontecer. Depois, uma breve introdução ao TCO para no final implementarmos o pattern Trampoline. 

## O problema

Temos a função `showCountDown` que exibe regressivamente todos os números a partir do número fornecido:

```javascript
const showCountDown = counter => {
  while(counter >= 0) {
    console.log(counter--);
  }
};
showCountDown(3);
/* exibe
3
2
1
*/
```
Podemos resolver o mesmo problema através de recursão, assunto que veremos a seguir.

## Um pouco sobre recursão

É possível refatorar a função `showCountDown` para uma versão recursiva. Vejamos:

```javascript
const showCountDown = counter => {
  if (counter < 0) return;
  console.log(counter);
  showCountDown(--counter);
};
showCountDown(3); // mesmo comportamento
```

*Uma função recursiva nada mais é do que uma função que chama a si mesma*. Todavia, é preciso haver um *leave event* para indicar o fim da recursão retornando um valor ou não. Em no caso, a instrução `if (counter < 0) return;` realiza esse papel. 

Que tal um exemplo de uma chamada recursiva que no fim deve retornar um valor? É o que veremos a seguir.

## Chamada recursiva retornando um valor

Um exemplo clássico de chamada recursiva que retorna um valor é o bem batido, mas nem por isso menos importante, cálculo <a href="https://pt.wikipedia.org/wiki/Fatorial" target="_blank">fatorial</a>. Vejamos um exemplo:

```javascript
const factorial = n => {
    // fatorial de 0 é 1!
    if(n <=1) return 1
    // return aqui!
    return n * factorial(--n);
};
console.log(factorial(3)) // 6;
```
>*É importante que o leitor resolva mentalmente a chamada recursiva anterior. O entendimento adquirido será a chave para compreender TCO e o pattern Trampolin.*

No código anterior, uma chamada à `factorial(3)` realizará mais duas chamadas à função `factorial`. Mas como isso parece na *call stack*? Aliás, o que seria essa call stack?

>*A **call stack** é composta de um ou mais **stack frames**. Cada stack frame corresponde a uma chamada para uma função que ainda não terminou, isto é, que ainda não retornou.*

Cada chamada recursiva adiciona um *stack frame* à pilha até chegar ao *leave event*. Então, cada *stack frame* começa a ser removida à medida que cada chamada retorna seus resultados. Visualmente temos algo assim:

```javascript
factorial(1) // 3 chamada
factorial(2) // 2 chamada
factorial(3) // 1 chamada
```

Quando `factorial(1)` for chamado, o *leave event* retornará `1` terminando a chamada recursiva. Isso fará com que cada stack frame seja removido da pilha:

```javascript
factorial(1) // 3 chamada / primeira a ser removida
factorial(2) // 2 chamada / segunda a ser removida
factorial(3) // 1 chamada / terceira a ser removida, retornando o fatorial
```

Porém, há um limite máximo de *stack frames* na *call stack* que ao ser excedido fará o programa terminar abruptamente.

## Ultrapassando o tamanho máximo da call stack

Vamos voltar ao exemplo do cálculo de um número fatorial, só que desta vez passaremos como argumento da função o valor `20000`:

```javascript
// código anterior omitido
console.log(factorial(20000)); 
// Uncaught RangeError: Maximum call stack size exceeded
```

Passamos do limite suportado pela call stack, isto é, estouramos a pilha de execução do nosso programa! Podemos fazer o mesmo com a função `showCountDown`:

```javascript
// código anterior omitido
showCountDown(20000);
// Uncaught RangeError: Maximum call stack size exceeded
```

Se as engines JavaScript suportassem *tail call optimization* (TCO), poderíamos organizar nosso código de uma maneira que evitaria o estouro da call stack. Aliás, sua implementação foi proposta no ES2015 (ES6) porém abandonada temporariamente por <a href="https://hipsters.tech/evolucao-e-especificacao-do-javascript-moderno/" target="_blank">questões de segurança que ainda precisam ser remediadas</a>. Mas o que seria esse TCO?

## Tail Call Otimization

TCO ocorre quando <a href="http://2ality.com/2014/04/call-stack-size.html" target="_blank">uma função é a última ação dentro de uma função. Quando esse requisito é atendido, a chamada para a própria função é feita via jump e não por meio de uma chamada de subrotina</a>.

Vejamos um exemplo que estoura a *call stack* em um *engine* JavasScript sem TCO:

```javascript
const fn = () => fn();
fn();
// Uncaught RangeError: Maximum call stack size exceeded
``` 
Dentro de um *engine* com suporte a TCO, o mesmo trecho de código será processado infinitamente sem estourar a *call stack*.

Outro exemplo é nossa função `factorial`. Ela precisa ser alterada para tirar benefício do TCO, pois possui como retorno a expressão `n * factorial(n - 1)` e não a chamada de uma função. 

A função `factorial`, adequada à TCO fica assim:

```javascript
// acc é o acumulador do resultado
const factorial = (acc, n) => {
    if(n <=1) return acc;
    // agora retorna a chamada da função
    return factorial(acc * n, --n);
};
console.log(factorial(1, 3)) // 6;
```

A boa notícia é que mesmo sem as `engines` JavaScript suportarem TCO, podemos evitar o estouro da *call stack* por meio de recursão através do **pattern Trampoline**.

## Trampolining pattern

O *Trampolining pattern* ou *trampoline* (trampolim, em português) é um loop que invoca funções que envolvem outras funções. 

>*Uma função que envolve outra função é chamada de **thunk**.* 

A função `trampoline` é implementada através de uma função que recebe outra função como argumento:

```javascript
// função que recebe outra como argumento
const trampoline = fn => {
/* implementação do trampoline */
};
```

Seu retorno é uma função que chamará repetidamente a função original recebida como parâmetro até que o retorno não seja um `thunk`:

```javascript
const trampoline = fn => {
  // faça enquanto for uma função
  while (typeof fn === 'function') {
      // chama a função repetidas vezes
      fn = fn();
  }
  // só retornada quando fn não for uma função!
  return fn;
};
```

Durante as chamadas sucessivas da função original, se algum valor que não for uma função for retornado, o `trampoline` parará imediatamente sua execução retornando o resultado da última chamada da função *fn* recebida como parâmetro.

Implementar a função `trampoline` não é suficiente.A função `factorial` que adequamos à TCO precisa retornar uma função (thunk) que ao ser invocada executará a operação recursiva:

```javascript
const factorial = (acc, num) => {
  if(num <= 1) return acc;
  // retorna uma função que ao ser chamada retorna 
  // a chamada de factorial
  return () => factorial(acc * num, --num);
}
```
Por fim, um teste que antes estourava a *call stack*:

```javascript
// infity, sem estourar a pilha
console.log(trampoline(factorial(1, 20000)));
```

A função `trampoline` existe apenas para controlar a execução de forma iterativa garantindo que haja apenas um *stack frame* em qualquer momento. Todavia, o código não executa tão rápido quanto sua versão recursiva, além de precisarmos ajustar a função para fazer uso do `trampoline`. 

Vejamos a função `showCountDown` modificada para ser utilizada por `trampoline`:

```javascript
const showCountDown = counter => {
  if (counter < 0) return;
  console.log(counter);
  return () => showCountDown(--counter);
};
// exibe o contador sem estourar a pilha
trampoline(showCountDown(20000));
```

## Conclusão

Por mais que TCO tenha sido proposto no ES2015 (ES6) sua implementação ainda não chegou nas engines dos navegadores do mercado, inclusive na plataforma Node.js. Porém, sua ausência pode ser contornada através do pattern Trampoline que exige uma ligeira modificação no código original.

E você? Já passou pelos problemas aqui descritos? Implementou ou utilizou alguma biblioteca para aplicar ao pattern trampoline? Deixe sua opinião.