

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

> app/controllers/sessions_controller.rb

    class SessionsController < ApplicationController
    
      def new
      end
    
      def create
        user = User.find_by(email: params[:session][:email].downcase)
        if user && user.authenticate(params[:session][:password])
          # Log the user in and redirect to the user's show page.
        else
          flash.now[:danger] = 'Invalid email/password combination'
          render 'new'
        end
      end
    
      def destroy
      end
    end

**Logging in**

> app/controllers/application_controller.rb

    class ApplicationController < ActionController::Base
      protect_from_forgery with: :exception
      include SessionsHelper
    end

> app/helpers/sessions_helper.rb

    module SessionsHelper
    
      # Logs in the given user.
      def log_in(user)
        session[:user_id] = user.id
      end
    end

> app/controllers/sessions_controller.rb

    class SessionsController < ApplicationController
    
      def new
      end
    
      def create
        user = User.find_by(email: params[:session][:email].downcase)
        if user && user.authenticate(params[:session][:password])
          log_in user
          redirect_to user
        else
          flash.now[:danger] = 'Invalid email/password combination'
          render 'new'
        end
      end
    
      def destroy
      end
    end

**Current user**

> app/helpers/sessions_helper.rb

    module SessionsHelper
      # Logs in the given user.
      def log_in(user)
        session[:user_id] = user.id
      end
    
    	# Returns the current logged-in user (if any).
      def current_user
      	# if @current_user.nil?
    		#   @current_user = User.find_by(id: session[:user_id])
    		# else
    		#   @current_user
    		# end
    		# @current_user = @current_user || User.find_by(id: session[:user_id])
        @current_user ||= User.find_by(id: session[:user_id])
      end
      
    end

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/login_success_mockup.png)


> app/helpers/sessions_helper.rb

    module SessionsHelper
    
      # Logs in the given user.
      def log_in(user)
        session[:user_id] = user.id
      end
    
      # Returns the current logged-in user (if any).
      def current_user
        @current_user ||= User.find_by(id: session[:user_id])
      end
    
      # Returns true if the user is logged in, false otherwise.
      def logged_in?
        !current_user.nil?
      end
    end

> app/views/layouts/_header.html.erb

    <header class="navbar navbar-fixed-top navbar-inverse">
      <div class="container">
        <%= link_to "sample app", root_path, id: "logo" %>
        <nav>
          <ul class="nav navbar-nav navbar-right">
            <li><%= link_to "Home", root_path %></li>
            <li><%= link_to "Help", help_path %></li>
            <% if logged_in? %>
              <li><%= link_to "Users", '#' %></li>
              <li class="dropdown">
                <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                  Account <b class="caret"></b>
                </a>
                <ul class="dropdown-menu">
                  <li><%= link_to "Profile", current_user %></li>
                  <li><%= link_to "Settings", '#' %></li>
                  <li class="divider"></li>
                  <li>
                    <%= link_to "Log out", logout_path, method: "delete" %>
                  </li>
                </ul>
              </li>
            <% else %>
              <li><%= link_to "Log in", login_path %></li>
            <% end %>
          </ul>
        </nav>
      </div>
    </header>

> app/assets/javascripts/application.js

    //= require jquery
    //= require jquery_ujs
    //= require bootstrap
    //= require turbolinks
    //= require_tree .


**Testando mudanças no layout**

vamos fazer um teste de integração com os seguintes passos:

- Visitar o login path.
- Postar informações válidas para o sessions path.
- Verificar se o link de login desapare.
- Verificar se o link de logout aparece.
- Verificar se o link de profile aparece.

> app/models/user.rb

    class User < ActiveRecord::Base
      before_save { self.email = email.downcase }
      validates :name,  presence: true, length: { maximum: 50 }
      VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
      validates :email, presence: true, length: { maximum: 255 },
                        format: { with: VALID_EMAIL_REGEX },
                        uniqueness: { case_sensitive: false }
      has_secure_password
      validates :password, presence: true, length: { minimum: 6 }
    
      # Returns the hash digest of the given string.
      def User.digest(string)
        cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                      BCrypt::Engine.cost
        BCrypt::Password.create(string, cost: cost)
      end
    end

> test/fixtures/users.yml

    michael:
      name: Michael Example
      email: michael@example.com
      password_digest: <%= User.digest('password') %>

> test/integration/users_login_test.rb

    require 'test_helper'
    
    class UsersLoginTest < ActionDispatch::IntegrationTest
    
      def setup
        @user = users(:michael)
      end
      .
      .
      .
      test "login with valid information" do
        get login_path
        post login_path, session: { email: @user.email, password: 'password' }
        assert_redirected_to @user
        follow_redirect!
        assert_template 'users/show'
        assert_select "a[href=?]", login_path, count: 0
        assert_select "a[href=?]", logout_path
        assert_select "a[href=?]", user_path(@user)
      end
    end

**Login automático ao se registrar**

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
    
      def show
        @user = User.find(params[:id])
      end
    
      def new
        @user = User.new
      end
    
      def create
        @user = User.new(user_params)
        if @user.save
          log_in @user
          flash[:success] = "Welcome to the Sample App!"
          redirect_to @user
        else
          render 'new'
        end
      end
    
      private
    
        def user_params
          params.require(:user).permit(:name, :email, :password,
                                       :password_confirmation)
        end
    end

> test/test_helper.rb

    ENV['RAILS_ENV'] ||= 'test'
    .
    .
    .
    class ActiveSupport::TestCase
      fixtures :all
    
      # Returns true if a test user is logged in.
      def is_logged_in?
        !session[:user_id].nil?
      end
    end

> test/integration/users_signup_test.rb

    require 'test_helper'
    
    class UsersSignupTest < ActionDispatch::IntegrationTest
      .
      .
      .
      test "valid signup information" do
        get signup_path
        assert_difference 'User.count', 1 do
          post_via_redirect users_path, user: { name:  "Example User",
                                                email: "user@example.com",
                                                password:              "password",
                                                password_confirmation: "password" }
        end
        assert_template 'users/show'
        assert is_logged_in?
      end
    end

**Loggind out** 

> app/helpers/sessions_helper.rb

    module SessionsHelper
    
      # Logs in the given user.
      def log_in(user)
        session[:user_id] = user.id
      end
      .
      .
      .
      # Logs out the current user.
      def log_out
        session.delete(:user_id)
        @current_user = nil
      end
    end

> app/controllers/sessions_controller.rb

    class SessionsController < ApplicationController
    
      def new
      end
    
      def create
        user = User.find_by(email: params[:session][:email].downcase)
        if user && user.authenticate(params[:session][:password])
          log_in user
          redirect_to user
        else
          flash.now[:danger] = 'Invalid email/password combination'
          render 'new'
        end
      end
    
      def destroy
        log_out
        redirect_to root_url
      end
    end

> test/integration/users_login_test.rb

    require 'test_helper'
    
    class UsersLoginTest < ActionDispatch::IntegrationTest
      .
      .
      .
      test "login with valid information followed by logout" do
        get login_path
        post login_path, session: { email: @user.email, password: 'password' }
        assert is_logged_in?
        assert_redirected_to @user
        follow_redirect!
        assert_template 'users/show'
        assert_select "a[href=?]", login_path, count: 0
        assert_select "a[href=?]", logout_path
        assert_select "a[href=?]", user_path(@user)
        delete logout_path
        assert_not is_logged_in?
        assert_redirected_to root_url
        follow_redirect!
        assert_select "a[href=?]", login_path
        assert_select "a[href=?]", logout_path,      count: 0
        assert_select "a[href=?]", user_path(@user), count: 0
      end
    end

