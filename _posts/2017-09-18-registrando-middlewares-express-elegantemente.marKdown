---
layout: post
title:  "Registrando middlewares no Express elegantemente"
description: .
date:   2017-09-18 07:30:00 -0300
categories:
permalink: /registrando-middlewares-express-elegantemente/
author: flavio_almeida
tags: [javascript, node, express, middleware]
image: logo.png
---
Registrar middlewares no <a href="http://expressjs.com/" target="_blank">Expres.js</a> é uma tarefa corriqueira para os desenvolvedores na plataforma Node.js. Será que podemos tornar esse processo menos verboso?

Vejamos o seguinte trecho de configuração de uma instância do Express:

```javascript
const express = require('express')
    , app = express()
    , cookieParser = require('cookie-parser')
    , session = require('express-session')
    , passport = require('passport');

// registrando cada middleware importado
app.use(cookieParser());
app.use(session(
	{ secret: 'homem avestruz', 
	  resave: true, 
	  saveUninitialized: true 
	}
));
app.use(passport.initialize());
app.use(passport.session());

// código posterior omitido
```

Para cada middleware registrado precisaremos chamar a função `app.use()`. Podemos simplificar um pouco esse processo através de uma função que se encarregará de registrar cada middleware para nós. Primeiro, vejamos a função isoladamente:

```javascript
const register = (...middlewares) => 
    app => middlewares.forEach(middleware =>
        app.use(middleware));
```
A função recebe um número indeterminado de parâmetros através do <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Functions/rest_parameters" target="_blank">Rest parameter</a>. Seu retorno é outra função que ao ser chamada espera receber uma instância do Express. A função percorrerá a lista de middlewares recebida por `register` e adicionará cada um deles na instância do express recebida. 

Veja a função em uso:

```javascript
const express = require('express')
    , app = express()
    , cookieParser = require('cookie-parser')
    , session = require('express-session')
    , passport = require('passport');

const register = (...middlewares) => 
    app => middlewares.forEach(middleware =>
         app.use(middleware));

register(
    cookieParser(),
    session(
        { secret: 'homem avestruz', 
          resave: true, 
          saveUninitialized: true 
        }
    ),
    passport.session(),
)(app);
// código posterior omitido
```

A função `register` conseguiu enxugar boa parte do código, evitando as múltiplas chamadas à `app.use()`. Inclusive, nada nos impede de insolarmos a função em um módulo para que possamos reutilizá-la em outros projetos.

## Conclusão

Muitas vezes nos acomodados ao que esse ou aquele framework nos oferece e esquecemos de aplicar um dos grandes poderes da programação é que a criatividade.