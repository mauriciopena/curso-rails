

8 - Login e Logout
------------------

    $ ./bin/rails generate controller Sessions new

> config/routes.rb

    Rails.application.routes.draw do
      root                'static_pages#home'
      get    'help'    => 'static_pages#help'
      get    'about'   => 'static_pages#about'
      get    'contact' => 'static_pages#contact'
      get    'signup'  => 'users#new'
      get    'login'   => 'sessions#new'
      post   'login'   => 'sessions#create'
      delete 'logout'  => 'sessions#destroy'
      resources :users
    end

**Login Form**


> app/views/sessions/new.html.erb

    <% provide(:title, "Log in") %>
    <h1>Log in</h1>
    
    <div class="row">
      <div class="col-md-6 col-md-offset-3">
        <%= form_for(:session, url: login_path) do |f| %>
    
          <%= f.label :email %>
          <%= f.email_field :email, class: 'form-control' %>
    
          <%= f.label :password %>
          <%= f.password_field :password, class: 'form-control' %>
    
          <%= f.submit "Log in", class: "btn btn-primary" %>
        <% end %>
    
        <p>New user? <%= link_to "Sign up now!", signup_path %></p>
      </div>
    </div>

**Buscando e autenticando um usuário**

> app/controllers/sessions_controller.rb

    class SessionsController < ApplicationController
    
      def new
      end
    
      def create
        user = User.find_by(email: params[:session][:email].downcase)
        if user && user.authenticate(params[:session][:password])
          # Log the user in and redirect to the user's show page.
        else
          # Create an error message.
          render 'new'
        end
      end
    
      def destroy
      end
    end

**Renderizando com uma mensagem flash**

    $ ./bin/rails generate integration_test users_login

> test/integration/users_login_test.rb

    require 'test_helper'
    
    class UsersLoginTest < ActionDispatch::IntegrationTest
    
      test "login with invalid information" do
        get login_path
        assert_template 'sessions/new'
        post login_path, session: { email: "", password: "" }
        assert_template 'sessions/new'
        assert_not flash.empty?
        get root_path
        assert flash.empty?
      end
    end




