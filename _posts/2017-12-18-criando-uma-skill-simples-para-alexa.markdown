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

1- Crie uma pasta para o projeto

2- No terminal entre na pasta e execute o seguinte comando:

{% highlight bash %}
pasta-projeto$ npm install alexa-app-server
{% endhighlight %}

3- Depois crie um arquivo chamado server.js e coloque o seguinte código:

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

4- Crie um arquivo chamado Procfile com o seguinte código

{% highlight bash %}
web: node server.js
{% endhighlight %}

5- Em seguida, crie uma pasta chamada apps, e dentro dela crie uma pasta com o nome da sua skill (sem espaços). Dentro dessa pasta execute os seguintes comandos:

{% highlight bash %}
pasta-projeto/apps/nome-skill/$ npm init
pasta-projeto/apps/nome-skill/$ npm install alexa-app –save
{% endhighlight %}

6- Depois crie um arquivo chamado index.js e coloque o seguinte código nele:

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

Esse código contem a skill que você está criando. Na linha onde tem `app.intent('sayHelloWorld')` fica o nome do seu intent. Você pode criar várias intents na mesma skill. As `utterances` são as formas de chamar aquela intent, e você pode colocar quantas achar necessário. O parametro passado para o método `response.say` é o que a alexa vai responder para o usuário. Você pode criar sua logica nesse trecho, podendo fazer chamadas para outros serviços ou funções que você tenha criado.

7- Com o serviço criado, nós colocaremos o serviço no servidor. Como mencionado acima, nesse tutorial estaremos usando o heroku, mas sinta-se a vontade pra usar qualquer servidor de sua escolha. Crie uma app no heroku e coloque o código que você acabou de criar nela. Existem várias formas de "subir" o seu código para o servidor, mas eu recomendo conectar com um repositório no Github. Para testar você deve abrir `https://nomdeApp.herokuapp.com/alexa/nome_skill`. Você deve ver algo como isso:

![Image da app no heroku]({{site.baseurl}}/assets/img/heroku-app-alexa.png)

Com isso o serviço da alexa skill está pronto. Agora temos que criar a skill na amazon.

### Criando Skill na Amazon

Para criar uma skill no site da amazon, você deve "logar" com a sua conta [nesse site](https://developer.amazon.com/edw/home.html#/skills) e clicar em `Add New Skill`, isso vai abrir uma página com um formulário para você colocar as informações da sua skill. Você deve preencher o formulário da seguinte maneira:

1. **Skill Type**: você deve selecionar `Custom Interaction Model`.
2. **Language**: você deve selecionar o idioma desejado.
3. **Name**: preencha com o nome da sua skill
4. **Invocation Name**: preencha com a forma que você deseja chamar sua skill na forma `Alexa ask nome da skill` (ex.: Alexa ask simple alexa skill). Isso irar abrir a skill.
5. Em seguida salve e vá para a próxima página.
6. **Intent Schema**: você deve preencher com o que aparece no campo `Schema` que aparece no seu serviço. Veja a image acima como exemplo.
7. **Sample Utterances**: preencha com o texto que aparece no campo `Utterances` que aparece no seu serviço. Veja a image acima como exemplo.
8. Aperte next, na próxima página em `Service Endpoint Type:` selecione HTTPS
9. No campo **Default** coloque o link do seu serviço (ex.: https://nomdeApp.herokuapp.com/alexa/nome_skill) e precione next
10. Para o certificado selecione `My development endpoint is a sub-domain of a domain that has a wildcard certificate from a certificate authority`, se você tiver usando o heroku. E pressione next.
11. Na próxima página pressione next de novo.
12. Nessa página preencha com os dados referentes a função da sua skill. Depois vá para a próxima página e faça o mesmo.
13. Ao completar, salve. Se você preencheu tudo corretamente, o botão `Submit for certification` deve está "clicavel".

Se a sua conta da amazon for a mesma do seu echo, a skill deve ser carregada para seu echo automaticamente. Para testar basta você falar "Alexa ask nomde skill" para abrir a skill, depois chame por sua intent usando uma das `utterances` cadastradas.