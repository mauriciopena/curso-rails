

9 - Editando, mostrando e excluíndo usuários
--------------------------------------------

mock do formulário de edição:

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/edit_user_mockup_bootstrap.png)

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
    
      def edit
        @user = User.find(params[:id])
      end
    
      private
    
        def user_params
          params.require(:user).permit(:name, :email, :password,
                                       :password_confirmation)
        end
    end


> app/views/users/edit.html.erb

    <% provide(:title, "Edit user") %>
    <h1>Update your profile</h1>
    
    <div class="row">
      <div class="col-md-6 col-md-offset-3">
        <%= form_for(@user) do |f| %>
          <%= render 'shared/error_messages' %>
    
          <%= f.label :name %>
          <%= f.text_field :name, class: 'form-control' %>
    
          <%= f.label :email %>
          <%= f.email_field :email, class: 'form-control' %>
    
          <%= f.label :password %>
          <%= f.password_field :password, class: 'form-control' %>
    
          <%= f.label :password_confirmation, "Confirmation" %>
          <%= f.password_field :password_confirmation, class: 'form-control' %>
    
          <%= f.submit "Save changes", class: "btn btn-primary" %>
        <% end %>
    
        <div class="gravatar_edit">
          <%= gravatar_for @user %>
          <a href="http://gravatar.com/emails" target="_blank">change</a>
        </div>
      </div>
    </div>

Repare que usamos novamente form_for(@user), idêntico ao formulário de cadastro, mas no formulário de edição o código fonte gerado é diferente:

    <form accept-charset="UTF-8" action="/users/1" class="edit_user"
          id="edit_user_1" method="post">
      <input name="_method" type="hidden" value="patch" />
      .
      .
      .
    </form>

o rails sabe se o formulário é para cadastrar novos usuários ou para editá-los usando o método new_record?: 

    $ rails console
    >> User.new.new_record?
    => true
    >> User.first.new_record?
    => false

vamos adicionar o link para editar o profile de usuário ao header:

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
                  <li><%= link_to "Settings", edit_user_path(current_user) %></li>
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


**Tentativas de edição mal sucedidas**

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
    
      def edit
        @user = User.find(params[:id])
      end
    
      def update
        @user = User.find(params[:id])
        if @user.update_attributes(user_params)
          # Handle a successful update.
        else
          render 'edit'
        end
      end
    
      private
    
        def user_params
          params.require(:user).permit(:name, :email, :password,
                                       :password_confirmation)
        end
    end


**Testando tentativas de edição mal sucedidas**

    $ ./bin/rails generate integration_test users_edit

> test/integration/users_edit_test.rb

    require 'test_helper'
    
    class UsersEditTest < ActionDispatch::IntegrationTest
    
      def setup
        @user = users(:michael)
      end
    
      test "unsuccessful edit" do
        get edit_user_path(@user)
        assert_template 'users/edit'
        patch user_path(@user), user: { name:  "",
                                        email: "foo@invalid",
                                        password:              "foo",
                                        password_confirmation: "bar" }
        assert_template 'users/edit'
      end
    end

**Tentativas de edição bem sucedidas (usando TDD)**

> test/integration/users_edit_test.rb

    require 'test_helper'
    
    class UsersEditTest < ActionDispatch::IntegrationTest
    
      def setup
        @user = users(:michael)
      end
      .
      .
      .
      test "successful edit" do
        get edit_user_path(@user)
        assert_template 'users/edit'
        name  = "Foo Bar"
        email = "foo@bar.com"
        patch user_path(@user), user: { name:  name,
                                        email: email,
                                        password:              "",
                                        password_confirmation: "" }
        assert_not flash.empty?
        assert_redirected_to @user
        @user.reload
        assert_equal name,  @user.name
        assert_equal email, @user.email
      end
    end


> app/controllers/users_controller.rb

    class UsersController < ApplicationController
      .
      .
      .
      def update
        @user = User.find(params[:id])
        if @user.update_attributes(user_params)
          flash[:success] = "Profile updated"
          redirect_to @user
        else
          render 'edit'
        end
      end
      .
      .
      .
    end

> app/models/user.rb

class User < ActiveRecord::Base
  before_save { self.email = email.downcase }
  validates :name, presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 }
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }, allow_nil: true
  .
  .
  .
end


**Authorization**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/login_page_protected_mockup.png)

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
      before_action :logged_in_user, only: [:edit, :update]
      .
      .
      .
      private
    
        def user_params
          params.require(:user).permit(:name, :email, :password,
                                       :password_confirmation)
        end
    
        # Before filters
    
        # Confirms a logged-in user.
        def logged_in_user
          unless logged_in?
            flash[:danger] = "Please log in."
            redirect_to login_url
          end
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
    
      # Logs in a test user.
      def log_in_as(user, options = {})
        password = options[:password] || 'password'    
        if integration_test?
          post login_path, session: { email:    user.email,
                                      password: password }
        else
          session[:user_id] = user.id
        end
      end
    
        private
    
        # Returns true inside an integration test.
        def integration_test?
          defined?(post_via_redirect)
        end
    end

> test/integration/users_edit_test.rb

    require 'test_helper'
    
    class UsersEditTest < ActionDispatch::IntegrationTest
    
      def setup
        @user = users(:michael)
      end
    
      test "unsuccessful edit" do
        log_in_as(@user)
        get edit_user_path(@user)
        .
        .
        .
      end
    
      test "successful edit" do
        log_in_as(@user)
        get edit_user_path(@user)
        .
        .
        .
      end
    end

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
      # before_action :logged_in_user, only: [:edit, :update]
      .
      .
      .
    end

> test/controllers/users_controller_test.rb

    require 'test_helper'
    
    class UsersControllerTest < ActionController::TestCase
    
      def setup
        @user = users(:michael)
      end
    
      test "should get new" do
        get :new
        assert_response :success
      end
    
      test "should redirect edit when not logged in" do
        get :edit, id: @user
        assert_not flash.empty?
        assert_redirected_to login_url
      end
    
      test "should redirect update when not logged in" do
        patch :update, id: @user, user: { name: @user.name, email: @user.email }
        assert_not flash.empty?
        assert_redirected_to login_url
      end
    end

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
      before_action :logged_in_user, only: [:edit, :update]
      .
      .
      .
    end

**Verificando se o usuário logado tem autorização**

> test/fixtures/users.yml

    michael:
      name: Michael Example
      email: michael@example.com
      password_digest: <%= User.digest('password') %>
    
    archer:
      name: Sterling Archer
      email: duchess@example.gov
      password_digest: <%= User.digest('password') %>

> test/controllers/users_controller_test.rb

    require 'test_helper'
    
    class UsersControllerTest < ActionController::TestCase
    
      def setup
        @user       = users(:michael)
        @other_user = users(:archer)
      end
    
      test "should get new" do
        get :new
        assert_response :success
      end
    
      test "should redirect edit when not logged in" do
        get :edit, id: @user
        assert_not flash.empty?
        assert_redirected_to login_url
      end
    
      test "should redirect update when not logged in" do
        patch :update, id: @user, user: { name: @user.name, email: @user.email }
        assert_not flash.empty?
        assert_redirected_to login_url
      end
    
      test "should redirect edit when logged in as wrong user" do
        log_in_as(@other_user)
        get :edit, id: @user
        assert flash.empty?
        assert_redirected_to root_url
      end
    
      test "should redirect update when logged in as wrong user" do
        log_in_as(@other_user)
        patch :update, id: @user, user: { name: @user.name, email: @user.email }
        assert flash.empty?
        assert_redirected_to root_url
      end
    end

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
      before_action :logged_in_user, only: [:edit, :update]
      before_action :correct_user,   only: [:edit, :update]
      .
      .
      .
      def edit
      end
    
      def update
        if @user.update_attributes(user_params)
          flash[:success] = "Profile updated"
          redirect_to @user
        else
          render 'edit'
        end
      end
      .
      .
      .
      private
    
        def user_params
          params.require(:user).permit(:name, :email, :password,
                                       :password_confirmation)
        end
    
        # Before filters
    
        # Confirms a logged-in user.
        def logged_in_user
          unless logged_in?
            flash[:danger] = "Please log in."
            redirect_to login_url
          end
        end
    
        # Confirms the correct user.
        def correct_user
          @user = User.find(params[:id])
          redirect_to(root_url) unless @user == current_user
        end
    end

> app/helpers/sessions_helper.rb

    module SessionsHelper
    
      # Logs in the given user.
      def log_in(user)
        session[:user_id] = user.id
      end
    
      # Returns true if the given user is the current user.
      def current_user?(user)
        user == current_user
      end
      .
      .
      .
    end

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
	    .
	    .
	    .
    
        # Confirms the correct user.
        def correct_user
          @user = User.find(params[:id])
          redirect_to(root_url) unless current_user?(@user)
        end
    end


