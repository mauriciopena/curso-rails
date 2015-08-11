
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


**3.5 Paginas ligeiramente dinamicas**

Vamos editar as páginas Home, Help, e About pages para que seu title seja diferente em cada uma delas, criar testes e refatorar no final para eliminar duplicação de código.

O resultado final deve ser que o título das páginas ficará no formato “nome da página | Curso de Ruby on Rails”, onde a primeira parte vai variar dependendo da página acessada.

    $ mv app/views/layouts/application.html.erb layout_file


**3.6 Testando title (vermelho)**

> test/controllers/static_pages_controller_test.rb

    require 'test_helper'
    
    class StaticPagesControllerTest < ActionController::TestCase
    
      test "should get home" do
        get :home
        assert_response :success
        assert_select "title", "Home | Curso de Ruby on Rails"
      end
    
      test "should get help" do
        get :help
        assert_response :success
        assert_select "title", "Help | Curso de Ruby on Rails"
      end
    
      test "should get about" do
        get :about
        assert_response :success
        assert_select "title", "About | Curso de Ruby on Rails"
      end
    end

**3.7 Adicionando títulos para as páginas (verde)**

> app/views/static_pages/home.html.erb

    <!DOCTYPE html>
    <html>
      <head>
        <title>Home | Curso de Ruby on Rails</title>
      </head>
      <body>
        <h1>Sample App</h1>
        <p>
          This is the home page.
        </p>
      </body>
    </html>



> app/views/static_pages/help.html.erb

    <!DOCTYPE html>
    <html>
      <head>
        <title>Help | Curso de Ruby on Rails</title>
      </head>
      <body>
        <h1>Sample App</h1>
        <p>
          This is the help page.
        </p>
      </body>
    </html>




> app/views/static_pages/about.html.erb

    <!DOCTYPE html>
    <html>
      <head>
        <title>About | Curso de Ruby on Rails</title>
      </head>
      <body>
        <h1>Sample App</h1>
        <p>
          This is the about page.
        </p>
      </body>
    </html>

**3.8 Layouts e Ruby inserido (Refatorar)**

> app/views/static_pages/home.html.erb

    <% provide(:title, "Home") %>
    <!DOCTYPE html>
    <html>
      <head>
        <title><%= yield(:title) %> | Curso de Ruby on Rails</title>
      </head>
      <body>
        <h1>Sample App</h1>
        <p>
          This is the home page.
        </p>
      </body>
    </html>

> app/views/static_pages/help.html.erb

    <% provide(:title, "Help") %>
    <!DOCTYPE html>
    <html>
      <head>
        <title><%= yield(:title) %> | Curso de Ruby on Rails</title>
      </head>
      <body>
        <h1>Sample App</h1>
        <p>
          This is the help page.
        </p>
      </body>
    </html>


> app/views/static_pages/about.html.erb

    <% provide(:title, "About") %>
    <!DOCTYPE html>
    <html>
      <head>
        <title><%= yield(:title) %> | Curso de Ruby on Rails</title>
      </head>
      <body>
        <h1>Sample App</h1>
        <p>
          This is the about page.
        </p>
      </body>
    </html>

Para eliminar a duplicação de código vamos restaurar nosso arquivo de layout:

    $ mv layout_file app/views/layouts/application.html.erb
    

> app/views/layouts/application.html.erb

    <!DOCTYPE html>
    <html>
      <head>
        <title><%= yield(:title) %> | Curso de Ruby on Rails</title>
        <%= stylesheet_link_tag    'application', media: 'all',
                                                  'data-turbolinks-track' => true %>
        <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
        <%= csrf_meta_tags %>
      </head>
      <body>
        <%= yield %>
      </body>
    </html>

Agora podemos limpar nossas páginas:

> app/views/static_pages/home.html.erb

    <% provide(:title, "Home") %>
    <h1>Sample App</h1>
    <p>
      This is the home page.
    </p>

> app/views/static_pages/help.html.erb

    <% provide(:title, "Help") %>
    <h1>Sample App</h1>
    <p>
      This is the help page.
    </p>

> app/views/static_pages/about.html.erb

    <% provide(:title, "About") %>
    <h1>Sample App</h1>
    <p>
      This is the about page.
    </p>


