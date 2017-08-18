---
layout: post
title:  "Por que você deveria estar usando util.promisify"
date:   2017-08-14 08:00:00 -0300
categories:
permalink: /por-que-voce-deveria-estar-usando-util-promisify/
author: flavio_almeida
tags: [node, javacript, promise, boas-praticas]
image: chamada.png
---

A estrela da vez é o Node.js! Este post parte do princípio de que o cangaceiro ou cangaceira já tenha um conhecimento, ainda que básico, do uso de Promises.

## Error first-callback

Quem desenvolve há alguns anos na plataforma Node.js com certeza já aplicou diversas vezes o padrão *error first-callback* adotado pelas API's assíncronas da plataforma. Vejamos um exemplo que cria um arquivo assíncronamente, inclusive você pode testá-lo em sua máquina:

```javascript
// importa a função writeFile do módulo fs
const { writeFile } = require('fs');

// recebe o nome do arquivo, conteúdo e o callback 
// para lidar com a operação 
writeFile('arquivo.txt', 'conteúdo do arquivo', err => {
  
  // verifica se houve algum erro
  // Se houve, logada e aborta a função
  if(err) return console.log(err);

  // se não houve erro
  console.log('arquivo criado com sucesso!');
}); 
```

No entanto, não é raro o desenvolvedor querer lidar com operações assícronas como a que acabamos de ver utilizando <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Promise" target="_blank">Promises</a>.

## Promisificando a operação de escrita

Podemos *promisificar* o código anterior da seguinte maneira:

```javascript
const { writeFile } = require('fs');

function criaArquivo(nome, conteudo) {

  return new Promise((resolve, reject) => {

    writeFile('arquivo.txt', 'conteúdo do arquivo', err => {
      if(err) return reject(err);
        resolve();
      }); 		
    });
}

criaArquivo()
  .then(() => console.log('arquivo criado com sucesso!'))
  .catch(err => console.log(err));
```

Excelente, mas termos que criar uma função auxiliar para converter uma API *error first callback* do Node.js para uma *Promise* pode ser um tanto tedioso. Uma solução seria alterar toda a API assíncrona do Node.js para trabalhar com Promises, mas isso quebraria toda a plataforma. Porém, a versão 8.0 do Node.js trouxe uma solução elegante para o problema.

## A magia de util.promisify

Para melhorar a experiência do deenvolvedor foi introduzida na versão 8.0 do Node.js a função utilitária `util.promisify`. Vejamos o código anterior com este novo recurso:

```javascript
// importou promisify do módulo util
const { promisify } = require('util');
const { writeFile } = require('fs');

// promisifica a função writeFile
const writeFilePromisificado = promisify(writeFile);

writeFilePromisificado('arquivo.txt', 'conteúdo arquivo')
  .then(() => console.log('arquivo criado com sucesso!'))
  .catch(err => console.log(err));
```

Ou, para tornar o código ainda mais enxuto:

```javascript
const { promisify } = require('util');

// realizou o require já promisificando
const writeFile = promisify(require('fs').writeFile);

writeFile('arquivo.txt', 'conteúdo arquivo')
  .then(() => console.log('arquivo criado com sucesso!'))
  .catch(err => console.log(err));
```

Com `util.promisify` podemos transformar qualquer API *error first callback* em Promises sem termos a responsabilidade de criar um wrapper para isso. É win, win win!

## Bônus

Com a transformação de API's assíncronas do Node.js em Promises através de `util.promisify` podemos utilizar sem grande burocracia a sintaxe `async/await`:

```javascript
const { promisify } = require('util');

// faz o require já promisificando
const writeFile = promisify(require('fs').writeFile);

(async function() {
  try {
    await writeFile('arquivo.txt', 'conteúdo arquivo');
    console.log('arquivo criado com sucesso!');
  } catch(err) {
    console.log(err);
  }
})();
```

# Conclusão

O `util.promisify` é um recurso recente na plataforma Node.js que permite transformar suas API's legadas em API's que retornem `Promises` com um mínimo de impacto na plataforma.