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

O Travis é um serviço de integração contínua que permite realizar diversos procedimentos toda vez que um determinado repositório, presente no Github, sofre alteração. Para realizar a integração entre o Github e o Travis basta seguir os passsos presentes nesse [link](https://docs.travis-ci.com/user/getting-started/). De forma sucinta, é necessário executar os seguintes passos:

* Crie uma conta no Travis e no Github
* No Travis, solicite ao Github que o Travis tenha acesso aos repositórios do Github 
* No Travis, selecione o repositório do Github que seja ser "Vigiado" pelo Travis
* No Git, crie um arquivo .travis.yml, como segue

```
language: python
python:
  - '3.3'

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
A tag **language** e **python** descrevem, respectivamente, qual é a linguagem que o projeto foi construido e qual versão do python foi utilizado. Essa informações são importantes, pois o Travis irá construir uma máquina virtual especifica para essa configuração. No caso do nosso projeto, será criada uma máquina linux com o python 3.3 instalada.
```
language: python
python:
  - '3.3'
```

As tags **branches** e **only** definem quais braches do GitHub que o Travis deve "vigiar". Em outras palavras, o travis será ativiado somente quando as branches desenvolvimento e master foram atualizadas.

```
branches:
  only:
  - master
  - desenvolvimento
```

A tag **install** é utilizado para instalar dependências necessárias para a compilação e testes do software em desenvolvimento. No nosso caso, utilizamos essa tag para instalar o Django e outras biblioteca. 

```
install:
  - pip install -r requirements.txt  
```

Por fim, a tag **script** é utilizada para executar comandos após a instalação das dependências. No nosso caso, usamos essa tag para definir a etapa de execução dos testes, ou seja, utiilzamos a tag script para automatizar a execução dos nossos testes.

```
script:
  - cd wsgi/myproject
  - python manage.py behave  
```

### Automatizando o Teste de Qualidade com o Travis e SonarQube

Todos estamos preocupados com a questão da Qualidade (e.g., processos, produtos, artefatos) dos projetos que estamos envolvidos. Aqui vamos explicar como podemos automatizar o controle da qualidade de código com o SonarQube e o Travis-CI:

* Crie uma conta no [SonarQube](https://sonarqube.com/)
* Crie uma chave privada no SonarQube com o seguintes passos: 
  ** My Account
  ** Security
  ** crie o token e guarde em algum lugar.
  

### Automatizando a Comunicação da equipe com o Travis e Slack

## Terceira Etapa: Delivery Contínuo - Entregando toda hora um software novo

### Automatizando a entrega do produto com Travis e OpenShift

##Referências:

**[Tutorial de BDD com Python, Django e Behave](https://semaphoreci.com/community/tutorials/setting-up-a-bdd-stack-on-a-django-application)**

**[How to Build and Deploy OpenShift Java Projects using Travis CI](https://blog.openshift.com/how-to-build-and-deploy-openshift-java-projects-using-travis-ci/)**

**[Integração Contínua e o processo Agile](http://blog.caelum.com.br/integracao-continua/)**

**[Integrando Github com Travis](https://docs.travis-ci.com/user/getting-started/)**
