---
layout: post
title:  "Funções velozes com o Memoization Pattern"
description: Memoization é uma técnica de otimização que consiste no cache do resultado de uma função baseada nos parâmetros de entrada. Neste artigo veremos como implementá-la na linguagem JavaScript e verificar na prática seus benefícios.
date: 2018-01-22 07:00:00 -0300
categories:
permalink: /funcoes-velozes-com-memoization-pattern/
author: flavio_almeida
tags: [javascript, memoization, memoisation, pattern, cache, functional]
image: logo.png
---
Memoization é uma técnica de otimização que consiste no cache do resultado de uma função baseada nos parâmetros de entrada. Neste artigo veremos como implementá-la na linguagem JavaScript e verificar na prática seus benefícios.

# O problema 

Irresistível não usar o exemplo do cálculo fatorial para fins didáticos. Vejamos:

```javascript
const factorial = n => {
    if(n <= 1) return 1
    return n * factorial(--n);
};
```

É uma função recursiva que não adere ao TCO, inclusive podendo estourar a pilha de execução dependendo do número recebido como parâmetro. O leitor pode saber mais consultando o artigo, <a href="http://cangaceirojavascript.com.br/limites-recursao-javascript-tco-e-pattern-trampoline/" target="_blank">"Limites da recursão em JavaScript, TCO e o pattern Trampoline"</a> deste mesmo autor.

Voltando para nossa função `factorial`. Vamos calcular o fatorial de **5** para logo em seguida calcularmos o fatorial de **3**:

```javascript
console.log(factorial(5)) // 120
console.log(factorial(3)) // 6
```

Excelente, porém uma observação minuciosa verificará que ao calcularmos o fatorial de 5 já calculamos o fatorial de 3. Que tal se a função `factorial` guardasse em algum lugar os números fatoriais já calculados? Nesse sentido, a chamada de `factorial(3)` buscaria desse tal lugar o resultado sem ter que calculá-lo. Uma solução para nosso problema é a aplicação do *pattern* **Memoization**.

## O pattern Memoization

*Memoization* é uma técnica de otimização que consiste no cache do resultado de uma função, baseado nos parâmetros de entrada.

>*Memoization é uma derivação da palavra `memorandum` que em latin significa "ser lembrado". Alguns chamam de Memoisation Patten utilizando a letra s no lugar de z.*

Para compreendermos esse pattern, vamos trabalhar com um exemplo de escopo menor, uma função que soma dois números:

```javascript
const sumTwoNumbers = (num1, num2) => num1 + num2;

console.log(sumTwoNumbers(5,5));
console.log(sumTwoNumbers(7,2));
console.log(sumTwoNumbers(3,3));
// repetiu a operação!
console.log(sumTwoNumbers(5,5)); 
```

Nossa função `sumTwoNumbers` é uma função **pura** (*pure function*). Funções puras são aquelas que retornam os mesmos valores para os mesmos parâmetros. Essa característica de uma função pura é chamada de *referential transparency*.

Queremos que a última chamada de `sumTwoNumbers(5,5)` não seja calculada, pelo contrário, queremos obter o resultado de um *cache* pois ele já foi calculado anteriormente. Para isso, vamos implementar a função `memoizer`, aquela que aplicará o *pattern* Memoization:

Apesar de ainda não termos implementado a função `memoizer`, quando pronta, ela será utilizada dessa maneira:

```javascript
const sumTwoNumbers = (num1, num2) => num1 + num2;
const memoizedSumTwoNumbers = memoizer(sumTwoNumbers);

console.log(memoizedSumTwoNumbers(5,5));
console.log(memoizedSumTwoNumbers(7,2));
console.log(memoizedSumTwoNumbers(3,3));
// a combinação de parâmetros já foi usada antes,
// por isso obterá o valor do cache
console.log(memoizedSumTwoNumbers(5,5)); // busca do cache
```

Agora que já sabemos o que esperar da nossa função `memoizer`, vamos implementá-la.

## Implementando a função memoizer

Nossa função `memoizer` receberá uma função (fn) como parâmetro e retornará outra função, aquela que potencialmente poderá ser chamada múltiplas vezes. Todavia, a função retornada precisará receber um número indefinido de parâmetros que será passado para a função (fn) que desejamos aplicar o `memoizer`. Conseguimos isso facilmente através de <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Functions/rest_parameters" target="_blank">parâmetros REST</a>:

Vejamos a estrutura inicial:

```javascript 
// recebe uma função como parâmetro
const memoizer = fn => {

    // retorna uma função que recebe 
    // um número indefinido de parâmetros
    return (...args) => {

        // usou REST operator para tratar todos 
        // os parâmetros como um array
    };
};
```

Precisamos de um local para guardar os resultados computados e para isso utilizaremos um `Map`:


```javascript 
const memoizer = fn => {

    const cache = new Map();

    return (...args) => {
        
    };
};
```

Toda vez que a nova função retornada por `memoizer` for chamada, ela precisará verificar se os parâmetros recebidos já existem em nosso `Map`. Os parâmetros serializados  (convertidos em texto) da função representam a `key` do nosso Map. Uma vez existindo, retornaremos o valor guardado evitando assim a execução da função `fn`. Se não existir, precisamos adicioná-lo em nosso `Map` e retornar o valor computado:

```javascript 
const memoizer = fn => {

    const cache = new Map();

    return (...args) => {
        // serializa o array args
        const key = JSON.stringify(args);

        // verifica se a chave existe
        if (cache.has(key)) {
            console.log(`Buscou do cache ${args}`);

            // retorna o valor do cache
            return cache.get(key);
        } else {
            console.log(`Não encontrou no cache ${args}. Adicionando ao cache.`);

            // invoca a função fn com os parâmetros
            // utilizando o spread operator
            const result = fn(...args);

            // guarda o resultado no cache
            cache.set(key, result);

            // retorna o valor que acabou de ser calculado
            return result;
        }
    };
};
```

Temos nossa função pronta. Vamos testá-la?

```javascript
// função memoizer omitida
const sumTwoNumbers = (num1, num2) => num1 + num2;

// turbina a função sumTwoNumbers
const memoizedSumTwoNumbers = memoizer(sumTwoNumbers);

console.log(memoizedSumTwoNumbers(5,5));
console.log(memoizedSumTwoNumbers(7,2));
console.log(memoizedSumTwoNumbers(3,3));

// a combinação de parâmetros já foi usada antes
console.log(memoizedSumTwoNumbers(5,5)); // busca do cache
```

Excelente! Nossa função funciona com qualquer tipo de parâmetro. Vejamos mais um teste:

```javascript
// recebe um objeto e um array
const getDiscount = (product, discounts) => discounts.get(product.id);

// descontos de um produto
const discounts = new Map();
discounts.set(1, 5);
discounts.set(2, 10);
discounts.set(3, 5);

// turbinou a função getDiscount
const memoizedGetDiscount = memoizer(getDiscount);

const result1 = memoizedGetDiscount(
    { id: 1, name: 'Cangaceiro JavaScript'}, 
    discounts
);
console.log(result1);

// buscou do cache
const result2 = memoizedGetDiscount(
    { id: 1, name: 'Cangaceiro JavaScript'}, 
    discounts
);
console.log(result2);
```

Que tal voltarmos para nossa função `factorial`?

## Memoizing da função factorial

Há um detalhe em nossa função `factorial` devido a sua sua natureza recursiva. Não podemos simplesmente aplicar a função `memoizer` como fizemos antes:

```javascript
const memoizedFactorial = memoizer(factorial);

console.log(memoizedFactorial(5));
console.log(memoizedFactorial(3));
console.log(memoizedFactorial(4));
```

Será exibido no console do Chrome:

```
Não encontrou no cache 5. Adicionando ao cache.
120
Não encontrou no cache 3. Adicionando ao cache.
6
Não encontrou no cache 4. Adicionando ao cache.
24
```

Apenas o resultado final de `factorial` entrará no cache. Então, quando chamarmos `memoizedFactorial(3)` o fatorial de 3 que já foi calculado pelo fatorial de 5 não estará no cache, muito menos o fatorial de 4.

Para contornarmos esse problema, vamos declarar nossa função `factorial` com auxílio da função `memoizer`:

```javascript
const factorial = memoizer(
    n => {
        if (n <= 1) return 1;
        return n * factorial(--n);
    }
);
console.log(factorial(5));
console.log(factorial(3));
console.log(factorial(4));
```

A saída no console do navegador será:

```
Não encontrou no cache 5. Adicionando ao cache.
Não encontrou no cache 4. Adicionando ao cache.
Não encontrou no cache 3. Adicionando ao cache.
Não encontrou no cache 2. Adicionando ao cache.
Não encontrou no cache 1. Adicionando ao cache.
120
Buscou do cache 3
6
Buscou do cache 4
24
```

Uma ligeira modificação, com um resultado bem interessante. 

Com base no que aprendemos, será que podemos realizar o cache do resultado de Promises? É o que veremos a seguir.

## Memoization de Promises

Temos a função `getNotaFromId()` com a finalidade de consumir uma API e retornar uma nota fiscal dado seu ID.

```javascript
// handler da resposta
const fetchHandler = res => 
    res.ok ? res.json() : Promise.reject(res.statusText)

// retorna uma Promise
const getNotaFromId = id =>
    fetch(`http://localhost:3000/notas/${id}`)
    .then(fetchHandler);
```

Se executarmos a operação duas vezes para um mesmo ID, acessaremos a API duas vezes.  Para termos certeza que uma função executará depois da outra, vamos encadear as duas chamadas:

```javascript
// código anterior omitido
getNotaFromId(1) // busca da API
    .then(console.log) 
    .then(getNotaFromId(1)) // busca da API
    .then(console.log)
    .catch(console.log);
``` 

A boa notícia é que podemos utilizar nossa função `memoizer` para realizar o cache do retorno da Promise:

```javascript
// código anterior omitido
const memoizedGetNotaFromId = memoizer(getNotaFromId);
memoizedGetNotaFromId(1) // busca da API
    .then(console.log) 
    .then(memoizedGetNotaFromId(1)) // busca do cache!
    .then(console.log)
    .catch(console.log);
```

Consultando o console do navegador vemos as seguintes mensagens emitidas pela função `memoizer`:

```
Não encontrou no cache 1. Adicionando ao cache.
Buscou do cache 1
Não encontrou no cache 2. Adicionando ao cache.
```

No entanto, há uma pegadinha. Se por acaso nossa `Promise` falhar por instabilidade na rede ou por qualquer outro problema, a função `memoizedGetNotaFromId()` fará o cache de uma Promise rejeitada.

Queremos permitir que o desenvolvedor possa, a qualquer momento, apagar o cache quando necessário. Para isso, vamos alterar ligeiramente a função `memoizer` que passará a oferecer a função `clearCache()`. Vamos aproveitar e eliminar as mensagem do console e simplificar ainda mais nossa função:

```javascript 
const memoizer = fn => {
    const cache = new Map();
    const newFn = (...args) => {
        const key = JSON.stringify(args);
        if(cache.has(key)) return cache.get(key);        
        const result = fn(...args);
        cache.set(key, result);
        return result;
    };
    newFn.clearCache = () => cache.clear();
    return newFn;
};
```

Realizando o teste:

```javascript
// código anterior omitido
const memoizedGetNotaFromId = memoizer(getNotaFromId);
memoizedGetNotaFromId(1) // busca da API
    .then(console.log) 
    .then(memoizedGetNotaFromId.clearCache()) // apagou o cache
    .then(memoizedGetNotaFromId(1)) // busca do API!
    .then(console.log)
    .catch(console.log);
```

Para que o leitor possa testar facilmente em sua máquina, segue um exemplo de uso da função `clearCache()` com a função `sumTwoNumbers`:

```javascript
// função memoizer omitida
const sumTwoNumbers = (num1, num2) => num1 + num2;
const memoizedSumTwoNumbers = memoizer(sumTwoNumbers);

console.log(memoizedSumTwoNumbers(5,5));
console.log(memoizedSumTwoNumbers(7,2));
console.log(memoizedSumTwoNumbers(3,3));
memoizedSumTwoNumbers.clearCache();
console.log(memoizedSumTwoNumbers(5,5)); // calcula novamente!
console.log(memoizedSumTwoNumbers(5,5)); // buscou do cache
```

Podemos recorrer à função `clearCache` sempre que necessário.

## Conclusão

O pattern Memoization permite acelerar a execução de funções cacheando seus resultados. Com suporte a *high order functions*, a linguagem JavaScript permite implementar o pattern com menos esforço.

E você? O que achou o pattern apresentado? Já precisou realizar o cache do resultado de métodos os funções? Qual estratégia utilizou? Deixe sua opinião para o Cangaceiro JavaScript aqui!
