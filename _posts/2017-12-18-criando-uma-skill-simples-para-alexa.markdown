---
layout: post
title: Criando uma Skill Simples para Alexa
date: 2017-12-18 13:20:20 -0300
description: 
img: 
tags: [Alexa Skills, Restfull, Self-hosted]
---
Neste tutorial você vai aprender como criar uma skill simples para alexa que tem dois intents: um para falar "Hello, world" e outra que diz a temperatura da cidade requisitada. Abaixo tem uma lista do que vai ser necessário para você criar sua primeira skill para Alexa. Neste tutorial nós iremos prover o serviço em um servidor rest, mas uma outra possibilidade é usar o serviço da [Amazon Web Services](https://aws.amazon.com) conhecido como lambda. 

### Considerações

Para criar esse tutorial usei algumas ferramentas que não são obrigatórias usar.

- Os códigos no terminal serão escritos considerando o sistema operacional MacOS;
- O serviço provido será usando node.js. Pode-se usar outros frameworks, desde que se tenha a biblioteca da alexa;
- Uma biblioteca para recuperar a temperatura chamada [weatherjs](http://weatherjs.com).

### Requisitos
- Conta na [Amazon](https://www.amazon.com);
- Um servidor para armazenar o serviço da skill. Nesse tutorial será usado o [Heroku](https://www.heroku.com) (ou outro servidor da sua escolha);
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

4- Crie um arquivo chamado Procfile com o seguinte código:

{% highlight bash %}
web: node server.js
{% endhighlight %}

5- Em seguida, crie uma pasta chamada apps, e dentro dela crie uma pasta com o nome da sua skill (sem espaços). Dentro dessa pasta execute os seguintes comandos:

{% highlight bash %}
pasta-projeto/apps/nome-skill/$ npm init
pasta-projeto/apps/nome-skill/$ npm install alexa-app –save
pasta-projeto/apps/nome-skill/$ npm install weather-js
{% endhighlight %}

6- Depois crie um arquivo chamado index.js e coloque o seguinte código nele:

{% highlight javascript %}
module.change_code = 1;
'use strict';

var weather = require('weather-js');
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

app.intent('sayWeather',
  {
    "slots" : {"city" : "AMAZON.US_CITY"},
    "utterances":[ 
        "what is the weather in {-|city}",
        "tell me the weather in {-|city}",
        "how is the weather in {-|city}"]
  },
  function(request,response) {
    var city = request.slot("city");
    return new Promise(function(resolve, reject) {
      weather.find({search: city, degreeType: 'C'}, (err, result) => {
        console.log("The city requested is " + city);
        if(err || result.length == 0) {
          response.say("The weather in " + city + " could not be found!").send();
          resolve();
        } else {
          var forecast = result[1].current.temperature;
          var location = result[0].location.name;
          response.say("The weather in " + location + " is " + forecast + " degrees celsius").send();
          resolve();
        }
      });
    });
  }
);6705,61


module.exports = app;
{% endhighlight %}

Esse código contem a skill que você está criando. Nas linhas onde tem `app.intent('sayHelloWorld',` e `app.intent('sayWeather',` ficam as suas intents. O método intent recebe três parâmetros. O primeiro é o nome da sua intent, o segundo é um dicionário que contém os `slots` e as `utterances` necessárias para sua intent, e o terceiro parâmetro é a função chamada quando sua intent for invocada. Você pode criar várias intents na mesma skill. Os `slots` são usados como variaveis. As `utterances` são as formas de chamar aquela intent, e você pode colocar quantas achar necessário. O parametro passado para o método `response.say` é o que a alexa vai responder para o usuário.

Nessa aplicação nós criamos duas intents uma que diz "Hello, world" e outra que diz a temperatura. Na primeira intent, sayHelloWorld, nós temos uma resposta simples, se alguma das utterances forem chamadas diga para o usuário "Hello, world". A segunda intent nós recuperamos o conteudo do slot em `var city = request.slot("city");`. Em seguida nós fazemos a requisição para a api de weather. Como essa é uma chamada assíncrona, nos temos que colocar a chamada em um promise para, ao final da requisição, a resposta ser enviada para a Alexa.

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

![Alexa skill primeira página]({{site.baseurl}}/assets/img/alexa-skill-page1.png)

6. **Intent Schema**: você deve preencher com o que aparece no campo `Schema` que aparece no seu serviço. Veja a image acima como exemplo.
7. **Sample Utterances**: preencha com o texto que aparece no campo `Utterances` que aparece no seu serviço. Veja a image acima como exemplo.

![Alexa skill segunda página]({{site.baseurl}}/assets/img/alexa-skill-page2.png)

8. Aperte next, na próxima página em `Service Endpoint Type:` selecione HTTPS
9. No campo **Default** coloque o link do seu serviço (ex.: https://nomdeApp.herokuapp.com/alexa/nome_skill) e precione next

![Alexa skill segunda página]({{site.baseurl}}/assets/img/alexa-skill-page3.png)

10. Para o certificado selecione `My development endpoint is a sub-domain of a domain that has a wildcard certificate from a certificate authority`, se você tiver usando o heroku. E pressione next.
11. Na próxima página pressione next de novo.

![Alexa skill segunda página]({{site.baseurl}}/assets/img/alexa-skill-page4.png)

12. Nessa página preencha com os dados referentes a função da sua skill. Depois vá para a próxima página e faça o mesmo.
13. Ao completar, salve. Se você preencheu tudo corretamente, o botão `Submit for certification` deve está "clicavel".

![Alexa skill segunda página]({{site.baseurl}}/assets/img/alexa-skill-page5.png)

Se a sua conta da amazon for a mesma do seu echo, a skill deve ser carregada para seu echo automaticamente. Para testar basta você falar "Alexa ask nomde skill" para abrir a skill, depois chame por sua intent usando uma das `utterances` cadastradas.
