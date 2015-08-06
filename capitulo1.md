
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

    $ sudo apt-get install -y libssl-dev libreadline-dev zlib1g-dev libffi-dev nodejs

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

instalar o bundler e depois rodar o reshash para atualizar os shims do binário do bundle

    $ gem install bundler
    $ rbenv rehash


----------

Criar um arquivo Gemfile com o sequinte conteúdo:

    source "https://rubygems.org"
    gem 'rails', '4.2.3'

Criar o esqueleto de uma aplicação:

    $ rails _4.2.0_ new testApp

pode ser preciso instalar o pocote de desenvolvimento do sqlite:

    $ sudo apt-get install libsqlite3-dev


Glossário:
----------

- Gems: é uma biblioteca, um conjunto de arquivos Ruby reusáveis
- Rails: framework composto por diversos pacotes (gems) escritos na linguagem ruby
- RubyGems: gerenciador de dependências padrão do ruby, vem instalado nas versões de ruby superiores a 1.9
- rbenv: gerenciador de versões de ruby
- Bundler: outro gerenciador de dependências de gems, mais robusto que o RubyGems, padrão do Rails
- shims: são executáveis que tem como única função passar comandos para o rbenv

**1.2.1 Ambiente de desenvolvimento**



1.3 Primeira aplicação
1.3.1 Bundler
1.3.2 rails server
1.3.3 Model-View-Controller (MVC)
1.3.4 Hello, world!
