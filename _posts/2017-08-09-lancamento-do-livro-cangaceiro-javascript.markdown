---
layout: post
title:  "Bastidores do Lançamento do livro Cangaceiro JavaScript"
date:   2017-08-09 09:18:51 -0300
categories: jekyll update
---

Finalmente! Foram 18 meses de gestação para que o "Cangaceiro JavaScript" chagasse até as mãos de vocês. Neste post, compatilharei as diretrizes que me norteram durante sua concepção. 

# Um livro que fosse além da sintaxe

Em mais de 500 páginas deste calhamaço fui da versão 5 do JavaScript até sua versão ES2017 (ES8). Fui além utilizando novos recursos que estão sendo especificados como <a href="https://github.com/tc39/proposal-decorators">Decorators</a> com auxílio de <a href="https://babeljs.io/">Babel</a>. Porém, desde a sua concepção, eu tinha em mente que ele não seria um livro focado única e exclusivamente na sintaxe da linguagem.

# Padrões de projeto e os paradigmas da Orientação a objetos e o Funcional

Eu queria um projeto do início ao fim que motivasse naturalmente a aplicação de padrões de projeto e o uso dos paradigmas da *Orientação a Objetos* e do *Funcional*, mas que ao mesmo tempo fosse simples. 

Durante a procura do ponto perfeito (bem que eu poderia participar MarterChef Brasil), acabei caricaturando recursos de frameworks populares do mercado. Digo "caricaturando", porque o livro nunca perdeu seu foco que é ensinar JavaScript transitando entre diferentes paradigmas e não a criação de um mini framework. Este acabou aparecendo como consequencia da aplicação de boas práticas e que foi surgindo naturalmente. Aliás, já existem muitos no mercado, não?

# Preparando o leitor para frameworks populares do mercado

A conseguencia das práticas anteriores acabaram favorecendo a curva de aprendizagem daqueles que desejam adotar frameworks <a href="http://hipsters.tech/single-page-applications-hipsters-16/" target="_blank">Single Page Application</a> do mercado como <a href="https://angular.io/" target="_blank">Angular</a> e bibliotecas para criação de componentes como <a href="https://facebook.github.io/react/">React</a>, mas que ainda não possuem os pré-requisitos necessários.

# Antecipando o Futuro

A decisão de incluir Babel, o transcompilador mais famoso do mundo open source permitiu que eu abordasse novos recursos que estão por vir na linguagem.


Por fim, a decisão de adicionar um capítulo dedicado exclusivamente ao Webpack foi de caso pensado. Por mais *vanilla* que o programador JavaScript seja, cedo ou tarde terá que se deparar com esse pedaço de tecnologia sofisticado utilizado por Angular CLI, React Create App, Vue CLI entre outros clientes de linha de comando. Configurar do zero Webpack em um projeto que não pertence a nenhum framework do mercado dará a confiança que muitos ainda querem atingir ao utilizá-lo.


