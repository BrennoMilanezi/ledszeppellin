# LedsZeppellin: Exemplificando a Integração Contínua e Entrega Contínuo do Leds

**Build Status**: [![Build Status](https://travis-ci.org/paulossjunior/ledszeppellin.png)](https://travis-ci.org/paulossjunior/ledszeppellin)

## Introdução

O **LedsZeppellin** é o nome dado ao conjunto de ferramentas, ou solução caso achem melhor, que o Leds utiliza implementar os conceitos de Integração contínua e Entregas Contínuas. Para o desenvolvimento desse tutorial utilizaremos as seguintes tecnologias: 

    Linguagem de Programação: Python
    Framework Web: Django
    Repositório de códigos: GitHub
    Framework de Behavior Driven Development: Behave
    Serviço de Integração Contínua: Travis
    Serviço de Deploy de aplicação: OpenShift
    Serviço de Controle de Versão: Github (
    Serviço de Qualidade de Código: Sonnar
    Serviçõ de Comunicação: Slack

### Pré-requisitos:

1. [Instalar e configurar o OpenShift e o Travis localmente](https://blog.openshift.com/how-to-build-and-deploy-openshift-java-projects-using-travis-ci/).
2. Instalar o Python 3.3.
3. [Instalar o Virtualenv](https://pythonhelp.wordpress.com/2012/10/17/virtualenv-ambientes-virtuais-para-desenvolvimento/).

## Etapa Zero: Um pouco de conceitos

>"Integração Contínua é uma pratica de desenvolvimento de software onde os membros de um time integram seu trabalho frequentemente, geralmente cada pessoa integra pelo menos diariamente – podendo haver multiplas integrações por dia. Cada integração é verificada por um ***build automatizado (incluindo testes)*** para detectar erros de integração o mais rápido possível. Muitos times acham que essa abordagem leva a uma significante redução nos problemas de integração e permite que um time desenvolva software coeso mais rapidamente." ***[Martin Fowler](http://martinfowler.com/articles/continuousIntegration.html) -  [Caelum](http://blog.caelum.com.br/integracao-continua/)***

## Primeira etapa: Construir uma aplicação com teste

A nossa aplicação será hospedada no **OpenShift**. O [OpenShift](https://www.openshift.com/) é um PASS (Platforma-as-a-Service) que permite aos desenvolvedores, de forma rápida, o desenvolvimento, hospedagem e a escala de suas aplicações em um ambiente cloud. 

Para criar a nossa aplicação no ambiente do OpenShift é necessário executar o seguinte comando:
    
    rhc app create ledszeppellin python-3.3 postgresql-9.2

Esse comando está informando ao OpenShift que desejamos criar uma aplicação com um o nome de **ledszeppellin** que use python 3.3 e postgresql 9.2 . Quando o criamos uma aplicação o OpenShift copia alguns arquivo para o máquina local. Esses arquivos não serão úteis para o nosso projeto. Assim, iremos apagá-los:

    cd ledszeppellin
    git rm -rf *
    git commit -am "deleted project"
   
O próximo passo é usarmos um template de projeto que utiliza Django e Python, disponibilizado pela equipe do OpenShift, para o nosso projeto.

    git remote add upstream -m master git://github.com/openshift/django-example.git
    git pull -s recursive -X theirs upstream master

Observe que agora temos uma aplicação configurada para funcionar dentro do OpenShift. Para realizar o teste da nossa aplicação, basta executar o seguinte comando:

    git push
Ao realizar o **push** no 
É importante comentar que o OpenShift trabalha de duas formas para gerenciar as dependências de biblioteas do projeto: (i) utilizando o arquivo de **setup.py** ou requirimentes.txt.  

    git remote rename origin openshift

## Segunda etapa: Integração Contínua - Qualidade ao Máximo 

### Integrando o Travis e Github

O Travis é um serviço de integração contínua que permite realizar diversos procedimentos, como testes automatizados, toda vez que um determinado repositório, presente no Github, sofre alteração. É um serviço totalmente customizável para se adequar a aplicação e sua linguagem usada. Para realizar a integração entre o Github e o Travis basta seguir os passos:
 
* Crie uma conta no Travis e no Github
* No Travis, solicite ao Github que o Travis tenha acesso aos repositórios do Github
* No Travis, selecione o repositório do Github que seja ser "Vigiado" pelo Travis
* No Git, crie um arquivo .travis.yml, como segue

Algumas coisas que o travis.yml pode conter:
 
* What programming language your project uses
* What commands or scripts you want to be executed before each build (for example, to install or clone your project’s dependencies)
* What command is used to run your test suite
* Emails, Campfire and IRC rooms to notify about build failures
 
Exemplo do arquivo travis.yml:
```
language: python
python:
  - '3.3'
dist: trusty
sudo: false

branches:
  only:
  - master
  - desenvolvimento

install:
  - pip install -r requirements.txt  

script:
  - cd wsgi/myproject
  - python manage.py behave 
```
Configuração do .travis.yml
 
Tag **language:**
A tag language e python descrevem, respectivamente, qual é a linguagem que o projeto foi construído e qual versão do python foi utilizado. Essa informações são importantes, pois o Travis irá construir uma máquina virtual específica para essa configuração.
 
https://docs.travis-ci.com/user/languages/python/
 
Tag **sudo:** e **dist:**
Se adicionar **false** a tag **sudo:** (**sudo: false**) estará dizendo ao Travis para usar a infraestrutura padrão, que é o Ubuntu 12.04.
    
Se adicionarmos a tag **dist: ** com o valor **trusty** (**dist: trusty**) estaremos dizendo ao Travis para usar a infraestrutura baseada em Ubuntu Linux Trusty 14.04, ou seja, uma versão LTS mais recente do Linux.
 
Se o valor de **sudo:** for **enable** (**sudo: enable**) estaremos dizendo ao Travis que iremos utilizar um ambiente mais customizável na máquina virtual. Por exemplo, podemos usar o **dist: trusty** junto ao **sudo: enable** para utilizar uma versão mais atualizada do Linux 14.04 na infraestrutura, ou utilizar a tag **os: osx** se precisar que a aplicação rode em um macOS.
 
https://docs.travis-ci.com/user/ci-environment/
 
Tag **branches:** e **only:**
As tags branches e only definem quais branches do GitHub que o Travis deve "vigiar". Em outras palavras, o travis será ativado somente quando as branches desenvolvimento e master foram atualizadas.
 
Tag **install:** e **script:**
A tag install é utilizado para instalar dependências necessárias para a compilação e testes do software em desenvolvimento. No nosso caso, utilizamos essa tag para instalar o Django e outras biblioteca, podendo especificar passo a passo, ou fornecendo o script (.sh) com os comandos necessários. Por fim, a tag script é utilizada para executar comandos após a instalação das dependências. No nosso caso, usamos essa tag para definir a etapa de execução dos testes, ou seja, utilizamos a tag script para automatizar a execução dos nossos testes. Assim como no install, os comandos podem ser passo a passo ou em um arquivo de script.

**Ciclo de preparação do ambiente:**
1- OPTIONAL Install apt addons
2- OPTIONAL Install cache components
3- before_install
4- install
5- before_script
6- script
7- OPTIONAL before_cache (for cleaning up cache)
8- after_success or after_failure
9- OPTIONAL before_deploy
10- OPTIONAL deploy
11- OPTIONAL after_deploy
12- after_script

Na pratica, nao existe diferenca caso escreva todos os passos de forma procedural em uma tag só, mas por motivos de organização e modularização de etapas, é preferível usar as várias tags que o Travis disponibiliza.
 
https://docs.travis-ci.com/user/installing-dependencies/


### Automatizando o Teste de Qualidade com o Travis e SonarQube

Todos estamos preocupados com a questão da Qualidade (e.g., processos, produtos, artefatos) dos projetos que estamos envolvidos. Nesses dois tutorais ([tutorial 1](https://trajano.net/2016/11/integrating-travis-sonarqube/) e [tutorial 2](https://docs.travis-ci.com/user/sonarqube/)) são apresentados as etapas para automatizar o controle da qualidade de código com o SonarQube e o Travis-CI.

Observer como o nosso travis foi configurado
```
addons:
    sonarqube:
        organization: "LEDS"
        token:
            secure: xxxxx

script:
  - cd wsgi/myproject
  - python manage.py behave
  - cd ..
  - cd ..
  - sonar-scanner
       
```
Por se tratar de um projeto em Python, torna-se necessário adicionar algum comando a mais na configuração da máquina que será criada. Isso ocorre, pois o SonnaQube foi desenvolvido em Java e torna-se necessário configurar o ambiente java. Para isso foi adicionado as seguintes linhas no .travis.yml:

```
jdk:
  - oraclejdk8
addons:
  apt:
    packages:
      - oracle-java8-installer 
before_script:
  - export JAVA_HOME=/usr/lib/jvm/java-8-oracle           

```

### Automatizando a Comunicação da equipe com o Travis e Slack
Como pode ser percebido, são diversos detalhes para que devemos estar atentos para o desenvolvimento de uma aplicação. Além disso, torna-se dificil acompanhar todos os acontecimentos de um projeto se não tivermos um canal único que centraliza a comunicação. Imagine trabalhar em um projeto com 10 programadores e tendo que acompanhar as novas versões dos código, as quebras de build, o controle de qualidade e outros fatores de um projeto de forma não automatizada. Um verdadeiro caos. Certo? 

Para resolver o problema acima, o travis permite enviar mensagens para os membros do projeto. Essa mensagens podem ser feitas por email ou por um aplicativo de comunicação com o Slack. Em nossa arquitetura utilizamos o slack e seguimos os seguintes tutoriais para a configuração do ambiente: [tutorial 1](https://docs.travis-ci.com/user/notifications/) e [tutorial 2](https://blog.travis-ci.com/2014-03-13-slack-notifications/). Veja um exemplo da nossa configuração:


```
notifications:
    slack: xxxxx
```
## Terceira Etapa: Delivery Contínuo - Entregando toda hora um software "novo"

### Automatizando a entrega do produto com Travis e OpenShift
Agora que definimos todas as etapas de integração contínua e como vamos realizar os testes automaticamente, que tal pensarmos em como fazermos deploy automaticamente também? Em outras palavras, que tal apenas realizarmos o commit na master e o Travis ficar responsável por "instalar o software no servidor" ?

No caso do Leds, utilizamos o OpenShift para ser o nosso servidor de deploy. Para saber como utilizá-lo integrado com o Travis leia esse [tutorial]((https://blog.openshift.com/how-to-build-and-deploy-openshift-java-projects-using-travis-ci/). Aqui está um exemplo da nossa integração:

```
deploy:
  provider: openshift
  user: paulossjunior@gmail.com
  password:
    secure: 1HM8jbE0223srglawzCMMJd2666ACV9nR5qOheRtmny8qQk1sw18Lh5Q6o+FHsGE1HcbP8FyJ4z+TXmQVRjU5vC9Jn2y3LcHQEIjvbuLxKobDPgokCXGNZiCBoCjBzZeXlLStIVLo4g4UvoIbLvVWpkyWImtdQaMtmg26povTLXSFS1v+IP/0Cvj+/wRz3he6Li4dd2d8OlKBgvt5UmNuo5loGtmkHqvqU+wdMJjCSXdzNkJFsLaBlra1iHgD2g/kIG4+pCWFA+WEgt8JLNuru6H5JEeoApA9HO8Oda332w5N55Yx1QSpM+EIzjnjooCfksiUgyrz90WttlF3i36fTwNt/P3SRVEXviFqui6/j7Sn8og1uMzHCpuN9jxl0MBeEi07rlukDqY5/AgXDrhoSaBRnZCWdgDyglS6B555CuUg4wlYyBIQgMAFrztWJ80+WE/6V48cgVmOE/99CUItnwkqYepwcyA+avadwz2QYcfOcevdWEuehQnqmWMQIamSfh47wlaip0WG6hrefKPNMT07z4j+xvBYdZaW4JHQmkblhS//QYmBzAd/wq9L+7eH50P4AE6wAa67aUXeakP+5CSFvb+STuFkqDqXJXu9nB6gJmQIgxIiM0Sa2wg4GoV77WKEbrn2vOqhfJCyFwcwP7ci0lJy/9W1GV0oQgtMnQ=
  app: ledszeppellin
  domain: paulossjunior
  on:
    repo: LEDS/ledszeppellin
all_branches: true
```

Com a configuração acima, a aplicação é instalada no servidor do OpenShift toda vez que não houver problemas nos testes ou na qualidade do código. 

That's all Folks.

## Outras Referências:

**[Tutorial de BDD com Python, Django e Behave](https://semaphoreci.com/community/tutorials/setting-up-a-bdd-stack-on-a-django-application)**

**[How to Build and Deploy OpenShift Java Projects using Travis CI](https://blog.openshift.com/how-to-build-and-deploy-openshift-java-projects-using-travis-ci/)**

**[Integração Contínua e o processo Agile](http://blog.caelum.com.br/integracao-continua/)**

**[Integrando Github com Travis](https://docs.travis-ci.com/user/getting-started/)**

**[Integrando Travis e Heroku](http://gabrielfeitosa.com/integracao-continua-com-travis-e-heroku/)**

**[Integrando Travis e Heroku 2](https://docs.travis-ci.com/user/deployment/heroku/)**
