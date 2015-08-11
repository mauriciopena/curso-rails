
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

    $ ./bin/rake test

**3.3 Vermelho**

> test/controllers/static_pages_controller_test.rb

    require 'test_helper'
    
    class StaticPagesControllerTest < ActionController::TestCase
    
      test "should get home" do
        get :home
        assert_response :success
      end
    
      test "should get help" do
        get :help
        assert_response :success
      end
    
      test "should get about" do
        get :about
        assert_response :success
      end
    end

**3.4 Verde**

> config/routes.rb

    Rails.application.routes.draw do
      get 'static_pages/home'
      get 'static_pages/help'
      get 'static_pages/about'
      .
      .
      .
    end
O teste continua falhando, mas a mensagem mudou. A rota que criamos está tentando encontrar uma action no static_pages_controller que não existe. Vamos criá-la:

> app/controllers/static_pages_controller.rb

    class StaticPagesController < ApplicationController
    
      def home
      end
    
      def help
      end
    
      def about
      end
    end

Ainda no vermelho, pois falta o template (a view)

    $ touch app/views/static_pages/about.html.erb

> app/views/static_pages/about.html.erb

    <h1>About</h1>
    <p>
      This is the sample application.
    </p>














