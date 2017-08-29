---
layout: post
title:  "Explicando currying em JavaScript no busão"
description: Final de semana passado peguei o mesmo busão que um aluno que começou na minha turma de Formação Front-End na Caelum. Durante o trajeto, meio acanhado ele me perguntou se eu poderia explicar para ele o conceito de currying em JavaScript através de um exemplo prático, pois ele teria uma prova na faculdade que cobraria o assunto.
date:   2017-08-28 13:00:00 -0300
categories:
permalink: /explicando-currying-javascript-busao/
author: flavio_almeida
tags: [javascript, currying, functional]
image: logo.png
---
Final de semana passado peguei o mesmo busão que um aluno que começou na minha turma de <a href="https://www.caelum.com.br/formacao-frontend" target="_blank">Formação Front-End na Caelum</a>. Durante o trajeto, meio acanhado ele me perguntou se eu poderia explicar para ele o conceito de **currying** em JavaScript através de um exemplo prático, pois ele teria uma prova na faculdade que cobraria o assunto. Para eu ter certeza que estávamos falando da mesma coisa, pedi que ele desse sua definição de currying. Sua resposta foi satisfatória:

>*"Ë uma técnica, uma maneira de transformar uma função que recebe múltiplos parâmetros de forma que ela pode ser chamada com um parâmetro apenas."*

Antes que eu pudesse dizer algo, ele pediu que o exemplo fosse bem simples, pois ele já estava quase chegando em seu destino. Eu tinha uma caneta (a que eu costumo assinar os certificados) e ele tinha um bloco de anotações. Isso já era o suficiente. Perguntei se no meu exemplo eu poderia usar *arrow function* e ele disse que não haveria problema algum. Então, eu tasquei o seguinte código no papel: 

```javascript
const ehDivisivel = (divisor, numero) => !(numero % divisor);
const divisor = 2;
console.log(ehDivisivel(divisor, 20)); // true
console.log(ehDivisivel(divisor, 11)); // false
console.log(ehDivisivel(divisor, 12)); // true
```

No código anterior, `ehDivisivel` é uma *function expression* que verifica se um número é divisível por outro, retornando `true` ou `false`, simples assim.

Perguntei qual era a minha intenção no contexto das instruções que escrevei. Ele foi rápido e disse:

>*"Sua intenção foi verificar se todos esses três números são divisíveis por dois"*

No final perguntei se ele conseguia enxergar alguma outra melhoria no código, mas ele disse que já estava tudo certo para ele. Foi então que comecei a instigá-lo. Foi mais ou menos assim:

>*Se a minha intenção nas três chamadas de `ehDivisivel` é utilizar o mesmo divisor, não é um tanto tedioso ter que ficar passando o divisor como parâmetro toda vez que eu quiser saber se um número é divisível por dois?*

Ele pensou e concordou. Perguntei como ele resolveria. Ele disse que bastaria alterar a função `ehDivisivel` e tornar o divisor um valor fixo. Disse para ele que isso não é solução, porque ainda queremos que `ehDivisivel` seja flexível para poder trabalhar com qualquer divisor. Enquanto ele pensava, rabisquei o seguinte código:

```javascript
const divisivelPor = divisor => numero => !(numero % divisor);
const ehDivisivel = divisivelPor(2);
console.log(ehDivisivel(20));
console.log(ehDivisivel(11));
console.log(ehDivisivel(12));
```

Caso o leitor ainda não esteja tão acostumado com *arrow function*, o mesmo código escrito utilizando `function` ficaria assim:

```javascript
/// versão sem arrow function
const divisivelPor = function(divisor) {
    return function(numero) {
        return !(numero % divisor);
    }
}
const ehDivisivel = divisivelPor(2);
console.log(ehDivisivel(20));
console.log(ehDivisivel(11));
console.log(ehDivisivel(12));
```

Assim que acabei de escrever o código dei início à explicação. 

Criei a *function expression* `divisivelPor` que recebe como parâmetro um **divisor** que desejamos utilizar e que tem como retorno uma função. A nova função recebe apenas **numero** como parâmetro e, ao ser chamada, levará em consideração o `divisor` recebido `divisivelPor` no cálculo realizado com o `numero` que recebeu. Isso só é possível graças ao suporte de **clojures** em JavaScript. Caso o leitor queira saber um pouco mais sobre este assunto pode conferir o meu post <a href="http://blog.alura.com.br/javascript-debounce-pattern-closure-e-duas-amigas/" target="_blank">JavaScript, debounce pattern, closure e duas amigas</a> no blog da Alura.

Antes que eu pudesse perguntar se ele entendeu, rapidamente ele disse:

>*A função retornada por divisivelPor que antes recebia dois parâmetros recebe um parâmetro apenas, simplificando a sua chamada em termos de parâmetros passados. Podemos usar divisivelPor para retornar uma nova função com outro divisor se assim desejarmos. Cara, muito obrigado*.

E então ele saiu correndo sem se despedir de mim, parece que perdeu a parada do ônibus mais próximo da sua casa. Ou será que se assustou com a explicação? Só saberei na próxima aula.

## Currying Vs Função Parcial?

Currying é uma função que recebe múltiplos parâmetros como entrada e retorna uma função com exatamente um parâmetro. Já funções parciais retornam uma função com menos parâmetros, ou seja, podemos reduzir uma função que recebe quatro parâmetros para uma função que receba duas.

 