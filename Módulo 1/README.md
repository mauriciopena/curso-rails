
1 - Instalação do ruby/rails
----------------------------
**1.1 Introdução**

 - Ruby on Rails é um framework de desenvolvimento web escrito na linguagem ruby. 
 - Foi lançado em 2004 e hoje é uma das ferramentas mais poderosas e populares para criação de aplicações web dinâmicas.
 - É 100% open-source, disponível sob a licença MIT.
 - Tarefas comuns em desenvolvimento web como gerar HTML, criar data models, e roteamento de URLs são fáceis com o Rails, e o código resultante da aplicação é conciso e de fácil entendimento.
 - Foi um dos primeiros frameworks a adotar a arquitetura RESTFUL.
 - Comunidade extremamente forte e participativa com centenas de contribuidores open-source, conferências anuais, um número enorme de gems, incontáveis blogs, fóruns, etc.
 
**1.2 Instalando o rbenv, ruby e Rails**

Instalar o git:

    $ sudo apt-get install git-core

----------

Instalar pacotes necessários:

    $ sudo apt-get install -y libssl-dev libreadline-dev zlib1g-dev libffi-dev nodejs libsqlite3-dev

----------


Instalar o rbenv via git:

    $ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv

Adicionar ~/.rbenv/bin ao $PATH

    $ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc

Adicionar o rbenv init ao shell para abilitar os shims.

    $ echo 'eval "$(rbenv init -)"' >> ~/.bashrc

Reiniciar o terminal para que as alterações no PATH façam efeito e depois verificar se o rbenv está funcionando:

    $ type rbenv
    => "rbenv is a function"

Opcional (mas recomendado), instalar o ruby-build: Instalando o ruby-build como um plugin do rbenv permite que seja utilizado o comando install do rbenv. Isso irá instalar a última versão de desenvovimento do ruby-build no diretório ~/.rbenv/plugins/ruby-build. Para atualizar o ruby-build rode um comando git pull para baixar as últimas alterações.

    $ git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build

Para listar as versões de ruby disponíveis:

    $ rbenv install -l

Para instalar um versão específica do ruby, utilize o comando install

    $ rbenv install 2.2.2

Instalar o plugin rbenv-bundler

    $ git clone https://github.com/carsomyr/rbenv-bundler.git ~/.rbenv/plugins/bundler

vamos criar uma pasta para nosso projeto:

    $ mkdir curso-rails

para setar o ruby deste projeto usamos o comando local do rbenv:

    $ rbenv local 2.2.2

depois podemos instalar o bundler utilizando o comando gem install do RubyGems 

    $ gem install bundler
 
como instalamos uma nova gem, precisamos rodar o comando rehash para atualizar os shims do binário do bundle

    $ rbenv rehash

----------

crie um arquivo Gemfile com o sequinte conteúdo na pasta do projeto:

    source "https://rubygems.org"
    gem 'rails', '4.2.3'

vamos utilizar o comando config do bundle para que definir o local onde as gems devem ser baixadas, nesse caso na pasta "vendor" do nosso projeto. 

    $ bundle config path vendor/bundle

depois rode o comando blundle install, isso irá baixar a gem especificada no arquivo Gemfile, no caso o rails 4.2.3, assim como todas as suas dependências. Não esquecer de rodar o comando rehash novamente no final.

    $ bundle install
    $ rbenv rehash

adicionar um gitignore para garantir que as gems baixadas não venham a parar no controle de versão

    $ echo 'vendor/bundle' >> .gitignore

Glossário:
----------

- Gems: é uma biblioteca, um conjunto de arquivos Ruby reusáveis
- Rails: framework composto por diversos pacotes (gems) escritos na linguagem ruby
- RubyGems: gerenciador de dependências padrão do ruby, vem instalado nas versões de ruby superiores a 1.9
- rbenv: gerenciador de versões de ruby
- Bundler: outro gerenciador de dependências de gems, mais robusto que o RubyGems, padrão do Rails
- shims: são executáveis que tem como única função passar comandos para o rbenv

**1.2.1 IDE**

http://www.sublimetext.com
https://packagecontrol.io/installation#st2

**1.3 Primeira aplicação**

    $ cd ~/curso_rails
    $ bundle exec rails _4.2.3_ new hello_app
    $ cd hello_app
    $ rails server

A estrutura básica de uma aplicação rails contém o seguinte:
   
**app/** Aqui fica o código do "Core" da aplicação, os models, as views,  os controllers e os helpers

**app/assets**	"assets"; (CSS), JavaScript e imagens

**bin/**	Arquivos binários executáveis

**config/**	Configurações gerais da aplicação

**db/** Arquivos do banco de dados

**doc/**	Documentação da aplicação

**lib/**	Módulos e pacotes de assets da aplicação

**lib/assets**	Pacotes de assets como CSS, JavaScript e imagens

**log/**	Arquivos de log da aplicação

**public/**	Dados públicos que podem ser acessados via browser, por exemplo páginas de erro

**bin/rails**	Binário que pode ser utilizado para geração de código, abrir sessões de console e rodar um servidor local

**test/**	Testes

**tmp/**	Arquivos temporários

**vendor/**	Código de terceiros como plugins e gems

**vendor/assets**	Pacotes de assets de terceiros como CSS, JavaScript e imagens

**README.rdoc**	Uma breve descrição da aplicação

**Rakefile**	Tarefas utilitárias disponíveis através do comando rake

**Gemfile**	Dependências de gems

**Gemfile.lock**	Lista de gems utilizadas atualmente pela aplicação

**config.ru**	Arquivo de configuração do Rack middleware

**1.3.1 Model-View-Controller (MVC)**
MVC é um padrão de arquitetura que impõe a separação entre a "lógica de negócio" e a interface gráfica (GUI) e lógica de apresentação. 
![padrão MVC](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/mvc_schematic.png)

**1.3.2 Hello, world!**

O objetivo dessa sessão é criar uma action de controller para renderizar a string "Hello, world!". Com isso poderemos substituir a página inicial padrão do rails com a página de hello world que iremos criar.

> *app/controllers/application_controller.rb*

    class ApplicationController < ActionController::Base
      # Prevent CSRF attacks by raising an exception.
      # For APIs, you may want to use :null_session instead.
      protect_from_forgery with: :exception
    
      def hello
        render text: "hello, world!"
      end
    end
  


> config/routes.rb


    Rails.application.routes.draw do
      # You can have the root of your site routed with "root"
      root 'application#hello'
    end





