---
layout: post
title:  "Limites da recursão em JavaScript, TCO e o pattern Trampoline"
description: Neste artigo aprenderemos a utilizar recursão sem medo na ausência de TCO através do pattern Trampoline.
date: 2018-01-15 06:00:00 -0300
categories:
permalink: /limites-recursao-javascript-tco-e-pattern-trampoline/
author: flavio_almeida
tags: [javascript, recursion, recursão, trampoline, functional programing, TCO]
image: logo.png
---
JavaScript é uma linguagem multiparadigma suportando os paradigmas da orientação a objetos e o funcional. Não é raro desenvolvedores mais voltados para o paradigma funcional se valerem de **recursão** para solucionar problemas. Porém, pela ausência de *tail call optimization* (**TCO**) nas engines do mercado, longas chamadas recursivas podem estourar a *call stack* terminando abruptamente o programa. Neste artigo aprenderemos a utilizar recursão sem medo na ausência de TCO através do *pattern* **Trampoline**. Primeiro, veremos um pouco sobre recursão e os problemas que podem acontecer. Depois, uma breve introdução a TCO para no final implementarmos o pattern Trampoline. 

## O problema

Vamos começar com um problema simples. Temos a função `showCountDown` que exibe regressivamente todos os números a partir do número fornecido:

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
Podemos resolver o mesmo problema através de recursão.

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

*Uma função recursiva nada mais é do que uma função que chama a si mesma*. Todavia, é preciso haver um *leave event* para indicar o fim da recursão retornando um valor ou não. Em nosso caso, a instrução `if (counter < 0) return;` realiza esse papel. 

Que tal um exemplo de uma chamada recursiva que no fim retorne um valor? É o que veremos a seguir.

## Chamada recursiva com retorno

Um exemplo clássico de chamada recursiva com retorno é o batido, mas nem por isso menos importante, cálculo <a href="https://pt.wikipedia.org/wiki/Fatorial" target="_blank">fatorial</a>. Vejamos um exemplo:

```javascript
const factorial = n => {
    // fatorial de 0 é 1!
    if(n <= 1) return 1
    // return aqui!
    return n * factorial(--n);
};
console.log(factorial(3)) // 6;
```
>*É importante que o leitor resolva mentalmente a chamada recursiva anterior. O entendimento adquirido será a chave para compreender a TCO e o pattern Trampoline.*

No código anterior, uma chamada à `factorial(3)` realizará mais duas chamadas à função `factorial`. Mas como isso aparece na *call stack*? Aliás, o que seria essa call stack?

>*A **call stack** (pilha de chamadas) é composta de uma ou mais **stack frames**. Cada stack frame corresponde a uma chamada para uma função que ainda não terminou, isto é, que ainda não retornou.*

Cada chamada recursiva adiciona uma *stack frame* à *call stack* até chegar ao *leave event*. A partir daí, começam a ser removidas à media que retornam seus resultados. Visualmente temos algo assim:

```javascript
factorial(1) // 3 chamada
factorial(2) // 2 chamada
factorial(3) // 1 chamada
```

Quando `factorial(1)` for chamado, o *leave event* retornará `1` terminando a chamada recursiva. Isso fará com que cada stack frame seja removida da pilha:

```javascript
factorial(1) // 3 chamada / primeira a ser removida
factorial(2) // 2 chamada / segunda a ser removida
factorial(3) // 1 chamada / terceira a ser removida, retornando o fatorial
```

Graficamente temos:

```bash
       ---|---
   ---|       |--- 
---               ---
```

Porém, há um limite máximo de *stack frames* na *call stack* que ao ser excedido fará o programa terminar abruptamente. Que tal irmos além da capacidade da *call stack* para vermos o que acontece?

## Ultrapassando o tamanho máximo da call stack

Vamos voltar ao exemplo do cálculo fatorial, mas desta vez utilizaremos `20000` como argumento da função:

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

E agora? Uma solução é adequarmos nossa chamada recursiva para que se benficie da **tail call optimization (TCO)**.

>*Alguns autores chamam de tail call elimination (TCE).*

Mas o que seria essa tal TCO e como ela pode nos ajudar? 

## Tail Call Optimization

A *tail call optimization* (TCO) ocorre quando o interpretador vê que a chamada de função recorrente é a **última** coisa na função a ser executada e **não há** nenhuma outra operação que precisa ser aplicada ao resultado retornado da função.  <a href="http://2ality.com/2014/04/call-stack-size.html" target="_blank">A chamada para a função corrente é feita via "jump" e não por meio de uma chamada de "subrotina"</a>. Em outras palavras, o interpretador otimiza a recursão eliminando a última chamada de função da call stack.

O exemplo abaixo, que estouraria a *call stack* em uma *engine* JavasScript **sem** TCO rodará indefinidamente sem provocar o estouro por uma engine **com** suporte a TCO:

```javascript
const fn = () => fn();
fn();
// -> Engine COM TCO: loop infinito
// -> Engine SEM TCO: Uncaught RangeError: Maximum call stack size exceeded
``` 

E nossa função `factorial`? Ela não se coaduna com a exigência da TCO, pois seu retorno é `n * factorial(--n);` e não a chamada de uma função apenas. 

A função `factorial`, adequada à TCO fica assim:

```javascript
// acc é o acumulador do resultado
const factorial = (acc, n) => {
    if(n <= 1) return acc;
    // agora retorna a chamada da função
    return factorial(acc * n, --n);
};
console.log(factorial(1, 3)) // 6;
```

Tudo muito bonito, só não é perfeito porque as engines JavaScript não suportam TCO! Como assim?

## Sem suporte a TCO nas engines JavaScript

A implementação da TCO faz parte do ES2015 (E6), porém os *browsers vendors* a deixaram de lado por questões de segurança que só apareceram durante a sua implementação. Você pode saber mais sobre este episódio no podcast  <a href="https://hipsters.tech/evolucao-e-especificacao-do-javascript-moderno/" target="_blank">"Evolução e Especificação do JavaScript Moderno", realizado por hipsters.tech</a>. 

A boa notícia é que mesmo sem as `engines` JavaScript suportarem TCO, podemos evitar o estouro da *call stack* por meio de recursão através do **pattern Trampoline**.

## Trampoline pattern 

O *pattern* Trampoline (trampolim, em português) é um loop que invoca funções que envolvem outras funções. 

>*Uma função que envolve outra função é chamada de **thunk**.* 

A função `trampoline` é implementada através de uma função que recebe outra função como argumento:

```javascript
// função que recebe outra como argumento
const trampoline = fn => {
/* implementação do trampoline */
};
```

A função trampoline chamará repetidamente a função `fn` recebida como parâmetro até que o retorno da função não seja um `thunk`:

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

Implementar a função `trampoline` não é suficiente. A função `factorial` que adequamos à TCO precisa retornar uma função (thunk) que ao ser invocada executará a operação recursiva:

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

>*O ato de converter uma função para suportar trampoline é chamado de **trampolining*** 
<div style="text-align: center">
<svg version="1.1" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
	 width="80px" height="80px" viewBox="0 0 300 300" enable-background="new 0 0 300 300" xml:space="preserve">
<path d="M88.905,31.075c15.268,16.733,30.992,33.047,47.068,49.005c5.43,4.957,10.269,10.647,16.457,14.709
	c2.811,1.884,6.404,2.786,9.692,1.687c1.923-0.64,4.644-1.095,4.969-3.543c1.52-3.735,1.068-8.328-1.699-11.373
	c-4.521-4.99-10.429-8.382-15.768-12.391c-11.003-7.731-22.502-14.71-33.597-22.3c-8.472-5.756-16.558-12.061-25.014-17.836
	c-1.419-0.624-2.8-2.279-4.481-1.489C86.291,29.105,88.172,29.901,88.905,31.075z"/>
<path d="M163.024,142.889c-2.615-1.785-6.257-1.731-8.837,0.08c-5.911,3.827-7.91,11.396-8.012,18.055
	c0.791,14.034,1.261,28.095,2.474,42.101c2.326,17.899,4.1,35.859,6.461,53.754c1.086,8.006,0.721,16.273,3.043,24.076
	c0.336-0.608,1.007-1.823,1.343-2.43c3.757-33.13,6.863-66.338,10.385-99.494c0.658-8.975,2.117-18.261-0.525-27.054
	C167.753,148.673,166.496,144.71,163.024,142.889z"/>
<path d="M240.507,261.649c-26.178,11.272-53.661,21.481-82.49,22.136c-20.968,0.52-41.565-5.066-60.784-13.072
	C63.511,256.737,32.236,237.735,0,220.77v6.931c16.751,9.403,33.704,18.452,50.603,27.59c23.177,11.83,46.498,24.173,71.95,30.354
	c16.362,4.4,33.586,5.088,50.369,3.143c18.129-2.281,35.748-7.62,52.685-14.359c25.576-9.735,50.02-22.117,74.393-34.487v-7.193
	C280.525,243.075,260.751,252.903,240.507,261.649z"/>
<circle cx="162.977" cy="50.317" r="22"/>
</svg>
</div>

A função `trampoline` existe apenas para controlar a execução de forma iterativa garantindo que haja apenas um *stack frame* em qualquer momento. Todavia, o código não executa tão rápido quanto sua versão recursiva, além de precisarmos ajustar a função para fazer uso do `trampoline`. 

Vejamos a função `showCountDown` modificada para ser utilizada por `trampoline`:

```javascript
const showCountDown = counter => {
  if (counter < 0) return;
  console.log(counter);
  // antes era "return showCountDown(--counter)",
  // agora retorna uma função
  return () => showCountDown(--counter);
};
// exibe o contador sem estourar a pilha
trampoline(showCountDown(20000));
```

## Conclusão

Por mais que a TCO tenha sido proposta no ES2015 (ES6) sua implementação ainda não chegou nas engines dos navegadores do mercado por ter criado brechas de seguança, inclusive na plataforma Node.js. Porém, sua ausência pode ser contornada através do pattern Trampoline que exige uma ligeira modificação no código original. Todavia, o uso do pattern não é tão performático quanto o suporte nativo à TCO.

E você? Já passou pelos problemas aqui descritos? Implementou ou utilizou alguma biblioteca para aplicar o pattern Trampoline? Deixe sua opinião.
