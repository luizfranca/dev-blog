---
layout: post
title: Criando uma Skill Simples para Alexa
date: 2017-12-18 13:20:20 -0300
description: 
img: 
tags: [Alexa Skills, Restfull, Self-hosted]
---
Neste tutorial você vai aprender como criar uma skill simples para alexa. Abaixo tem uma lista do que vai ser necessário para você criar sua primeira Amazon Skill para Alexa. Neste tutorial nós iremos prover o serviço em um servidor rest, mas uma outra possibilidade é usar o serviço da [Amazon Web Services](https://aws.amazon.com). 

### Considerações

Para criar esse tutorial usei algumas ferramentas que não são obrigatórias.

- Os códigos no terminal serão escritos considerando o sistema operacional MacOS;
- O serviço provido será usando node.js, pode-se usar outros frameworks. Desde que se tenha a biblioteca da alexa.

### Requisitos
- Conta na [Amazon](https://www.amazon.com);
- Um servidor para armazenar o serviço da skill, nesse tutorial será usado o [Heroku](https://www.heroku.com) (ou outro servidor da sua escolha);
- Node.js instalado;

### Serviço

A primeira parte necessária para criar uma Alexa Skill é criar o serviço que irá tratar as requisições feitas pelo usuário.

1. Crie uma pasta para o projeto
2. No terminal entre na pasta e execute o seguinte comando:

{% highlight bash %}
pasta-projeto$ npm install alexa-app-server
{% endhighlight %}

3. Depois crie um arquivo chamado server.js e coloque o seguinte código:

{% highlight javascript %}
'use strict';

var AlexaAppServer = require('alexa-app-server');

var server = new AlexaAppServer( {
	httpsEnabled: false,
	port: process.env.PORT || 3001
} );

server.start(); 
{% endhighlight %}

Esse código ira iniciar o servidor. Caso uma porta não seja fornecida pelo servidor ele irá usar a porta 3001.

4. Crie um arquivo chamado Procfile com o seguinte código

{% highlight bash %}
web: node server.js
{% endhighlight %}

5. Em seguida, crie uma pasta chamada apps, e dentro dela crie uma pasta com o nome da sua skill (sem espaços). Dentro dessa pasta execute os seguintes comandos:

{% highlight bash %}
pasta-projeto/apps/nome-skill/$ npm init
pasta-projeto/apps/nome-skill/$ npm install alexa-app –save
{% endhighlight %}

6. Depois crie um arquivo chamado index.js e coloque o seguinte código nele:

{% highlight javascript %}
module.change_code = 1;
'use strict';

var alexa = require('alexa-app');
var app = new alexa.app('hello_world');


app.launch( function(request, response) {
	response.say('Welcome to your alexa skill')
	.reprompt('Ask alexa to say hello world!')
	.shouldEndSession(false);
} );


app.error = function(exception, request, response) {
	console.log(exception)
	console.log(request);
	console.log(response);	
	response.say('Sorry an error occured ' + error.message);
};

app.intent('sayHelloWorld',
  {
    "utterances":[ 
		"say hello world",
		"tell me hello world"]
  },
  function(request,response) {
    response.say("Hello, world!");
  }
);

module.exports = app;
{% endhighlight %}

Esse código contem a skill que você está criando. Na linha onde tem app.intent('sayHelloWorld') fica o nome do seu intent. As utterances são as formas de chamar aquela intent, e você pode colocar quantas achar necessário. O response.say é o que a alexa vai responder para o usuário, você pode criar sua logica nesse trecho, podendo fazer chamadas para outros serviços ou funções que você tenha criado.