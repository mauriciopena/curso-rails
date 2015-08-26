

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




