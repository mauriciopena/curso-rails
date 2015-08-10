
3 - Páginas quase estáticas
--------

    $ cd ~/curso-rails
    $ bundle exec rails _4.2.3_ new sample_app

Vamos editar o Gemfile para ficar da seguinte maneira:

    source 'https://rubygems.org'
    
    gem 'rails', '4.2.3'
    gem 'sqlite3'
    gem 'sass-rails', '~> 5.0'
    gem 'uglifier', '>= 1.3.0'
    gem 'coffee-rails', '~> 4.1.0'
    gem 'jquery-rails'
    gem 'turbolinks'
    gem 'jbuilder', '~> 2.0'
    gem 'sdoc', '~> 0.4.0', group: :doc
    
    group :development, :test do
      gem 'byebug'    
      gem 'web-console', '~> 2.0'    
      gem 'spring'
    end
    
    group :test do
      gem 'minitest-reporters'
      gem 'mini_backtrace'
      gem 'guard'
      gem 'guard-minitest'
    end

**3.1 Gerando páginas estáticas**

    $ ./bin/rails generate controller StaticPages home help

**3.2 Primeiros testes**







