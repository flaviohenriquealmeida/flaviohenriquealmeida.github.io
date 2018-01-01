---
layout: post
title:  "Babel 7: configuração, uso de preset e de plugins"
description: A versão 7 do Babel esta prestes a ser lançada com diversas melhorias e novos recursos, inclusive o suporte à linguagem TypeScript. Neste artigo aprendemos a configurá-la e a utilizar plugins para adicionar recursos ainda propostos na linguagem JavaScript. 
date: 2018-01-01 06:00:00 -0300
categories:
permalink: /babel-7-configuracao-uso-presets-plugins/
author: flavio_almeida
tags: [babel 7, transpiler, javascript, typescript, babylon, compiler]
image: logo.png
---
Muitas vezes o desenvolvedor deseja utilizar o que há de mais moderno 
na linguagem JavaScript, mas se frustra pela ausência de suporte nos navegadores ou na plataforma Node.js. A boa notícia é que podemos gerar um código compatível através de <a href="https://babeljs.io/" target="_blank">Babel</a>, o compilador JavaScript mais utilizado pela comunidade. A versão 7 do Babel esta prestes a ser lançada com diversas melhorias e novos recursos, inclusive o suporte à linguagem TypeScript. Neste artigo aprendemos a configurá-la e a utilizar plugins para adicionar recursos ainda propostos na linguagem JavaScript.

## Pré-requsito

Para utilizarmos Babel precisamos do Node.js instalado. É altamente recomendável utilizar a última versão (par) disponível. Este artigo utilizou a versão 8.9.3. Existem várias maneiras de se instalar o Node.js, você pode consultá-las em <a href="https://nodejs.org/en/" target="_blank">https://nodejs.org/en/</a>.

>*Babel 7 não suporta mais as versões do Node.js 0.10, 0.12 e 5.*

Agora que já sabemos qual versão do Node.js utilizar podemos avançar para a estrutura do projeto.

## Estrutura do projeto

Nosso próximo passo será criar a pasta `project` e dentro dela a subpasta `app-src`. É dentro desta pasta que ficarão todos os arquivos do projeto, aqueles que serão compilados pelo Babel. Vamos aproveitar e criar também o arquivo `project/app-src/example.js`:

```bash
project
├── app-src
│   └── example.js
```
Dentro da pasta `project` criaremos o arquivo `package.json` através do comando:

```
npm init -y
```
Excelente, ficaremos com a seguinte estrutura:

```bash
project
├── app-src
│   └── example.js
└── package.json
```
>*Para quem nunca utilizou o Node.js, entenda o package.json como uma caderneta na qual ficam listados todos os módulos utilizados pela aplicação, inclusive scripts criados pelo desenvolvedor para automatizar tarefas*.

## Instalando Babel 7

Uma das mudanças da versão 6 para a versão 7 é o uso de **pacotes com escopo**. Pacotes com escopo são instalados através de `@escopoDomodulo/módulo`. 

>*Na data de publicação deste artigo, a versão do Babel utilizada foi a `7.0.0-beta.36`.* 

Instalamos os módulos `core` e `cli` desta forma:

```
npm install -S @babel/core @babel/cli
```

O `@babel/core` é o centro nervoso do babel. Já o `@babel/cli` é o cliente de linha de comando que nos permite interagir com o `@babel/core`. 

## Chamando babel-cli através de um script

Agora que já temos as dependências baixadas, precisamos criar um script que chame babel para nós indicando como alvo a pasta `app-src`.

Vamos alterar `project/package.json`:

```javascript
// código anterior omitido 
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "babel app-src -d app --source-maps",
    "watch": "babel app-src -d app --source-maps --watch"
  },
// código posterior omitido
```
Dentro do objeto atribuído à propriedade `"scripts"` adicionamos mais duas propriedades, a `"build"` e a `"watch"`. 

O primeiro script compilará todos os arquivos dentro de `app-src` toda vez que for executado, já o segundo monitorará em tempo real os arquivos da pasta `app-src` e, se algum arquivo for modificado, disparará o processo de compilação sem termos que nos preocupar com ele. Independente do script chamado, os arquivos resultantes do processo de compilação ficarão dentro da pasta `project/app`. 

>*O parâmetro `--source-maps` é importante, pois ele permitirá debugar o script gerado apontando a linha do erro no arquivo original e não no arquivo compilado.*

Vamos alterar o arquivo `project/app-src/example.js` e adicionar uma humilde chamada ao `console.log`:

```javascript
// project/app-src/example.js

console.log('Fui compilado pelo Babel 7, que máximo!');
```
Agora, dentro da pasta `project`, através do nosso terminal favorito, vamos executar o script `build` através do comando:

```bash
npm run build
```

A pasta `project/app` será criada e dentro dela teremos o arquivo `example.js` compilado:

```bash
project
├── app <-- pasta criada pelo Babel
│   └── example.js <-- arquivo compilado
├── app-src
│   └── example.js
└── package
```
O comando resultará no arquivo `project/app/example.js`, idêntico ao arquivo original em `project/app-src/example.js`. São idênticos porque ainda não utilizamos nenhum recurso que Babel nos oferece através de *presets* ou *plugins*, chegou a hora de instalá-los.

>*Quando usamos um compilador como Babel, são os scripts gerados que carregamos em nosso navegador ou em nossa aplicação Node.js, por isso que dizemos que a aplicação depende de um build step.*

## @babel/preset-env

O primeiro módulo que instalaremos é o `@babel/preset-env`. Ele é um módulo especial que compilará nosso código para uma versão anterior do ECMASCRIPT caso o recurso que estejamos utilizando não seja suportado pelos navegadores do mercado. 

Vamos instalá-lo:

```bash
npm install -S @babel/preset-env
```

Não basta instalá-lo, precisamos configurar Babel para que leve em consideração o módulo e fazemos isso através do arquivo `project/.babelrc`:

```bash
// project/.babelrc
{
    "presets": [
      ["@babel/preset-env", {
        "targets": {
          "browsers": ["last 2 versions"]
        }
      }]
    ],
}
```

>*Todos os presets prefixados com ES20xx foram descontinuados, sendo obrigatório o uso de @babel/preset-env*.

O código anterior adiciona `@babel/preset-env` como *preset* utilizado pelo Babel configurando-o ao mesmo tempo. Com essa configuração, apenas recursos que ainda não forem implementados nas duas últimas versões dos navegadores serão *transcompilados* para um código compatível.

>*Este preset se baseia em informações de compatibilidade publicadas em https://github.com/kangax/compat-table*

Agora que já temos o preset configurado, vamos dar um salto no futuro e utilizar recursos que foram propostos ao TC39, o comitê que rege o futuro da específicação ECMASCRIPT.

## Instalando plugins

Há uma série de plugins criados exclusivamente para Babel. Para efeito de teste, instalaremos dois deles que implementam propostas que ainda entrarão na linguagem, são eles:

* **@babel/plugin-proposal-pipeline-operator**: este autor já abordou o udo do pipeline operator em <a href="http://cangaceirojavascript.com.br/pipeline-operator-proposta-interessante-tc39/" target="_blank">outro</a> artigo.
* **@babel/plugin-proposal-optional-chaining**: forma elegante para nos proteger contra o acesso de propriedades de objetos nulas 

Vamos baixar os dois plugins de uma só vez:

```bash
npm install -S @babel/plugin-proposal-pipeline-operator @babel/plugin-proposal-optional-chaining 
```

Agora que já temos os plugins baixados, veremos na prática os recursos oferecidos por eles.

## @babel/plugin-proposal-pipeline-operator

O pipeline operator é essencialmente um açúcar sintático para funções que recebem um argumento apenas e que são encadeadas. 

Vamos alterar `product/app-src/example.js` adicionando o seguinte código:

```javascript
// product/app/example.js

const trim = text => text.trim();
const toUpperCase = text => text.toUpperCase();
const split = separator => text => text.split(separator);
const word = ' Calopsita do Agreste '

// com o pipeline operator
const words =  word 
  |> trim 
  |> toUpperCase 
  |> split(' ');
  
console.log(words); // ['CALOPSITA', 'DO', 'AGRESTE']
```

Após executarmos o comando `npm run build` o arquivo `product/app/example.js` ficará com essa estrutura:

```javascript
var _ref, _ref2, _word;

var trim = function trim(text) {
  return text.trim();
};

var toUpperCase = function toUpperCase(text) {
  return text.toUpperCase();
};

var split = function split(separator) {
  return function (text) {
    return text.split(separator);
  };
};

var word = ' Calopsita do Agreste ';
var words = (_ref = (_ref2 = (_word = word, trim(_word)), toUpperCase(_ref2)), split(' ')(_ref));
console.log(words);
var product = {
  name: 'Cangaceiro JavaScript',
  price: 29.90,
  type: 'book',
  author: {
    name: 'Flávio Almeida'
  }
};
//# sourceMappingURL=teste.js.map
```

Apesar de ser estruturalmente diferente do arquivo original, é um código que funcionará nos navegadores vigentes e na plataforma Node.js. 

## @babel/plugin-proposal-optional-chaining

Temos o seguinte código que acessa a propriedade de um objeto associado.

```javascript
// app-src/example.js

const product = {
    name: 'Cangaceiro JavaScript', 
    price: 29.90,
    type: 'book',
    author: {
        name: 'Flávio Almeida'
    }
}

const name = product.author.name;
console.log(name);
```

Excelente, mas se a propriedade `author` não existir? Podemos no blindar com o seguinte truque:

```javascript
const name = product.author && product.author.name;
console.log(name);
```

No entanto, com o uso do `optional-chaining` podemos simplificar nosso código para:

```javascript
const name = product?.author.name;
console.log(name);
```

Após compilarmos o arquivo, o resultado final será:

```javascript
// product/app/example.js

var product = {
  name: 'Cangaceiro JavaScript',
  price: 29.90,
  type: 'book',
  author: {
    name: 'Flávio Almeida'
  }
};
var name = product === null || product === void 0 ? void 0 : product.autor.name;
//# sourceMappingURL=teste.js.map
```
## Uma nota sobre Babel e TypeScript

Apesar de o TypeScript historicamente poder realizar (quando configurado) a compilação de um código escrito em ES2015 (ES6) para ES5 e suportar alguns recursos do ES2017 (ES8) como *async/await*, seu foco não é compilar recursos em desenvolvimento ou recém adicionados na especificação JavaScript que carecem implementação nos navegadores do mercado ou na plataforma Node.js. Nesse sentido, Babel é muito mais livre para implementar esses recursos.

A boa notícia é que a partir da <a href="https://github.com/prettier/prettier/issues/2334" target="_blank">contribuição do próprio time do TypeScript ao parser Babylon</a> utilizado pelo Babel e com o preset <a href="https://www.npmjs.com/package/babel-preset-typescript" target="_blank">babel-preset-typescript</a>, será possível utilizar todo o poder de Babel com esta linguagem. Programadores na linguagem TypeScript poderão antecipar o futuro e utilizar recursos mais modernos da linguagem JavaScript.

## Conclusão

Com auxilio de Babel podemos utilizar o que há de mais top na linguagem JavaScript sem termos que esperar o suporte chegar aos navegadores ou até mesmo na plataforma Node.js. Agora, com o parser do Babylon suportando TypeScript, programadores nesta linguagem também poderão se beneficiar do Babel.

E você? Já utiliza Babel? Compartilhe sua experiência conosco.
