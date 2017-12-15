---
layout: post
title:  "Barrels: facilitando importações de módulos"
description: O ECMAScript e a plataforma Node.js possuem seu próprio sistema de módulos. O primeiro utiliza o formato ESM, já o segundo o formato CommonJS. Todavia, independente do sistema utilizado, podemos simplificar bastante a importação de módulos através de Barrels, uma técnica, assunto deste artigo. 
date:   2017-12-12 07:00:00 -0300
categories:
permalink: /barrels-simplificando-importacoes-de-modulos/
author: flavio_almeida
tags: [javascript, module, barrel, ESM]
image: logo.png
---

O ECMAScript e a plataforma Node.js possuem seu próprio sistema de módulos. O primeiro utiliza o formato *ESM*, já o segundo o formato *CommonJS*. Todavia, independente do sistema utilizado, podemos simplificar bastante a importação de módulos através de **Barrels**, uma técnica, assunto deste artigo.

## O problema

O módulo `app` precisa importar uma série de artefatos de outros módulos para que possa ser implementado. Vejamos o trecho de código que realiza essas importações: 

```javascript 
// app/app.js

import { Negotiation } from './domain/negotiation';
import { Negotiations } from './domain/negotiations';
import { NegotiationService } from './domain/negotiation-service';
import { NegotiationsView } from './ui/negotiations-view';
import { NegotiationsView } from './ui/negotiation-view';

class NegotiationHandler {
    /* código omitido */
}
```

No código anterior, repetimos cinco vezes a instrução `import` para cada artefato importado. Importamos três deles de uma mesma pasta (entenda a pasta como um barril cheio de coisas), no caso `domain`. Os demais são importados da pasta `ui`. Se a aplicação fosse ainda mais complexa, teríamos uma explosão de instruções `import`. Todavia, podemos simplificar bastante a importação de módulos com a técnica **barrels** (barris).

## Criando barris

Dentro da pasta `app/domain`, vamos criar o arquivo `index.js`. Esse nome é importante, pois é o módulo padrão procurado quando utilizamos a instrução `import` apenas com o nome da pasta.

No módulo `index.js`, vamos exportar todos os artefatos de todos os módulos que fazem parte de `domain`:

```javascript
// app/domain/index.js

export * from './negotiation';
export * from './negotiations';
export * from './negotiation-service';
```

Criaremos um barril (barrel) também para `app/ui`:

```javascript
// app/ui/index.js

export * from './negotiation-view';
export * from './negotiations-view';
```

Nesse ponto, é importante entender que tanto `domain/index.js` quanto `ui/index.js` são pontos de entrada dos módulos que consolidam todos os artefatos que fazem parte deles. 

Voltando para o código inicial do artigo, podemos agora importar os artefatos da seguinte maneira:

```javascript 
// app/app.js

import { Negotiation, Negotiations, NegotiationService } from './domain';
import { NegotiationsView, NegotiationsView } from './ui';

class NegotiationHandler {
    /* código omitido */
}
```

Reduzimos bastante o número de instruções `import` e ainda conseguimos enxergar com clareza artefatos que fazem parte de um mesmo grupo. Aliás, podemos lançar mão da mesma estratégia com os módulos do CommonJS, do Node.js.

## Barris com CommonJS (Node.js)

Vamos pegar o mesmo exemplo utilizado no problema inicial com ESM, mas desta vez utilizando CommonJS:

```javascript 
// app/app.js

const Negotiation = require('./domain/negotiation')
    , Negotiations = require('./domain/negotiations')
    , NegotiationsService = require('./domain/negotiation-service')
    , NegotiationsView } require('./ui/negotiations-view')
    , NegotiationsView } require('./ui/negotiation-View');

class NegotiationHandler {
    /* código omitido */
};
```

Criando um barril para `domain`:

```javascript
// app/domain/index.js
const Negotiation = require('./negotiation')
    , Negotiations = require('./negotiations')
    , NegotiationsService = require('./negotiation-service');

module.exports = { 
    Negotiation, Negotiations, NegotiationsService 
};
```
Agora, para `ui`:

```javascript
// app/ui/index.js

const NegotiationsView = require('./ui/negotiations-view')
    , NegotiationsView = require('./ui/negotiation-view');

module.exports = { NegotiationsView, NegotiationsView };
```

E por fim, importando os artefatos, inclusive usando `destructuring`. Aliás, este autor já escreveu sobre <a href="http://cangaceirojavascript.com.br/importando-modulos-do-nodejs-com-destructuring/" target="_blank">como importar módulos do Node.js com destructuring</a>.

```javascript 
// app/app.js

const { Negotiation, Negotiations, NegotiationService } = require('./domain');
const { NegotiationsView, NegotiationsView } = require('./ui');
    
class NegotiationHandler {
    /* código omitido */
};
```

## Conclusão

Independente se usamos ESM ou CommonJS, inclusive ambos, podemos simplificar bastante a importação de módulos com a técnica de *barrels*. Melhorar e legibilidade e manutenção do nosso código é uma meta que não tem fim.

E você, o que achou dessa técnica? Já a conhecia antes? Deixe seu comentário.