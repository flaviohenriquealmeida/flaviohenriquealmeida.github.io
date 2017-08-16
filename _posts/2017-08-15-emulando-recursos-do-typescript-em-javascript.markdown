---
layout: post
title:  "Emulando recursos do TypeScript em JavaScript"
date:   2017-08-15 08:30:00 -0300
categories:
permalink: /emulando-recursos-do-typescript-em-javascript/
author: flavio_almeida
---

O objetivo desse post é demonstrar como o TypeScript pode servir como fonte de inspiração para escrevermos um código menos verboso e fácil de manter em JavaScript. Começaremos primeiro com uma visão geral sobre a linguagem TypeScript.

## Sobre a linguagem TypeScript

O <a href="https://www.typescriptlang.org/" target="_blank">TypeScript</a> é um superset do JavaScript que fornece principalmente tipagem estática opcional, classes e interfaces. Um dos grandes benefícios é permitir que IDEs forneçam um ambiente mais rico para detectar erros comuns à medida que escrevemos nosso código.

Agora que já temos uma ideia desta linguagem, vejamos dois recursos interessantíssimos que ela oferece. O primeiro deles é um atalho para definição de propriedades de classe.

## Atalho para definição de propriedades da classe

Em TypeScript, temos a seguinte declaração de classe:

```javascript
class Conta {

  private _titular: string;
  private _banco: string;
  private _agencia: string;
  private _numero: string;

  constructor(
    titular: string, 
    banco:string, 
    agencia: string, 
    numero: string) {

    this._titular = titular;
    this._banco = banco;
    this._agencia = agencia;
    this._numero = numero;
  }
  // métodos acessadores e alteradores omitidos
}
```
Cada parâmetro recebido no `constructor` equivale aos valores das propriedades `this._titular`, `this._banco`, `this._agencia` e `this._numero`. O programador terá que atribuir cada parâmetro na propriedade equivalente. Todavia, é possível simplificar a declaração desta forma:

```javascript
class Conta {

  constructor(
    private _titular: string, 
    private _banco: string, 
    private _agencia: string,
    private _numero: string) {}

  // métodos acessadores e alteradores omitidos
}
```
Muito mais enxuta não? 

Vejamos a mesma classe utilizando JavaScript:

```javascript
class Conta {

  constructor(titular, banco, agencia, numero) {
    this._titular = titular;
    this._banco = banco;
    this._agencia = agencia;
    this._numero = numero;
  }
  // métodos acessadores e alteradores omitidos  
}
```
Em JavaScript, não há modificadores de acesso muito menos tipagem estática. Porém, será que podemos simplificar o `constructor` desta classe para escrevermos menos como na versão utilizando TypeScript? Sim, podemos:


```javascript
class Conta {

  constructor(_titular, _banco, _agencia, _numero) {

    Object.assign(this, { _titular, _banco, _agencia, _numero});
  }
  // métodos acessadores e alteradores omitidos  
}
```
A solução acima utilizou dois recursos da linguagem. O primeiro, `Object.assign` que permite criar um objeto ou modificar um já existente. No exemplo anterior, estamos modificando a própria instância da classe `Conta` passando `this` como primeiro parâmetro para `Object.assign`. 

O segundo parâmetro é um objeto no formato literal utilizando uma sintaxe abreviada que terá suas propriedades copiadas para a instância da classe `Conta`. A sintaxe literal abreviada é possível quando a propriedade do objeto tem o mesmo nome da variável que contém o valor que a propriedade receberá. 

Em suma, a declaração abreviada 

```
Object.assign(this, { _titular, _banco, _agencia, _numero});
```

Equivale a declação não abreviada:

```
Object.assign(this, { 
    _titular: _titular, 
    _banco: _banco, 
    _agencia: _agencia,
    _numero: _numero});
```

Não conseguimos uma sintaxe tão enxuta como no TypeScript, mas ao tentarmos perseguir sintaxe semelhante acabamos simplificando nosso código JavaScript.

Vejamos outro recurso do TypeScript a seguir.

## Parâmetros obrigatórios

Ainda no contexto da classe `Conta` na versão em TypeScript, somos obrigados a passar todos os parâmetros do `constructor` ou teremos um erro de compilação:

```javascript
class Conta {

  constructor(
    private _titular: string, 
    private _banco: string, 
    private _agencia: string,
    private _numero: string) {}

  // métodos acessadores e alteradores omitidos  
}

// não compila
// Não passou o restante dos parâmetros do constructor
const conta = new Conta('Flávio'');  
```

Em JavaScript, podemos conseguir comportamento semelhante fazendo um uso não usual de `default parameter`, parâmetro padrão. O valor padrão no `constructor` da classe será uma função que lançará sempre uma exceção. Nesse contexto, se qualquer parâmetro for omitido, um erro em tempo de execução (runtime) será exibido:

```javascript
function obrigatorio(campo) {
  
  throw new Error(`${campo} é obrigatório`);
}

class Conta {

  constructor(
      _titular= obrigatorio('titular'), 
      _banco= obrigatorio('banco'), 
      _agencia=  obrigatorio('agência'),
      _numero= obrigatorio('número')
      ) {

      Object.assign(this, { _nome, _sobrenome, _idade});
  }
  // métodos acessadores e alteradores omitidos
}

// lança a exceção!
const conta = new Conta('Flávio');
```

Diferente do TypeScript que exibiria um erro em tempo de compilação a solução anterior exibirá um error em *rutime*, ou seja, apenas quando nossa aplicação estiver rodando. No entanto, é uma maneira interessante de tornarmos obrigatórios os parâmetros do `constructor` da classe com menos esforço.

## Conclusão

Não é possível implementar os recursos do TypeScript que vimos neste post diretamente na linguagem JavaScript, todavia eles e outros recursos podem servir de inspiração para que desenvolvedor JavaScript escreva um código cada vez melhor.

Esses e outros truques você encontra no livro <a href="https://www.casadocodigo.com.br/products/livro-cangaceiro-javascript">Cangaceiro JavaScript: Uma aventura no sertão da programação</a>.