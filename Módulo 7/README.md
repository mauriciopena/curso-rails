

7 - Cadastro de Usuário
-----------------------

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/profile_mockup_profile_name_bootstrap.png)

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/profile_mockup_bootstrap.png)

**Debug e Rails environments**

> app/views/layouts/application.html.erb

    <!DOCTYPE html>
    <html>
      .
      .
      .
      <body>
        <%= render 'layouts/header' %>
        <div class="container">
          <%= yield %>
          <%= render 'layouts/footer' %>
          <%= debug(params) if Rails.env.development? %>
        </div>
      </body>
    </html>

O Rails tem três ambientes configurados disponíveis: test, development, e production. O ambiente padrão do Rails console é o development:

      $ ./bin/rails console
      Loading development environment
      >> Rails.env
      => "development"
      >> Rails.env.development?
      => true
      >> Rails.env.test?
      => false

É possível rodar o console com em ambiente diferente, basta passar um parâmetro:

      $ ./bi/rails console test
      Loading test environment
      >> Rails.env
      => "test"
      >> Rails.env.test?
      => true

Assim como no console, development é o ambiente padrão do Rails server, mas também pode ser iniciado com outro ambiente:

    $ ./bin/rails server --environment production

Para que a aplicação funcione em modo produção é preciso criar o banco de dados:

      $ ./bin/rake db:migrate RAILS_ENV=production


> app/assets/stylesheets/custom.css.scss

    @import "bootstrap-sprockets";
    @import "bootstrap";
    
    /* mixins, variables, etc. */
    
    $gray-medium-light: #eaeaea;
    
    @mixin box_sizing {
      -moz-box-sizing:    border-box;
      -webkit-box-sizing: border-box;
      box-sizing:         border-box;
    }
    .
    .
    .
    /* miscellaneous */
    
    .debug_dump {
      clear: both;
      float: left;
      width: 100%;
      margin-top: 45px;
      @include box_sizing;
    }

**Users resource**

> config/routes.rb

    Rails.application.routes.draw do
      root             'static_pages#home'
      get 'help'    => 'static_pages#help'
      get 'about'   => 'static_pages#about'
      get 'contact' => 'static_pages#contact'
      get 'signup'  => 'users#new'
      resources :users
    end

> app/views/users/show.html.erb

    <%= @user.name %>, <%= @user.email %>


> app/controllers/users_controller.rb

    class UsersController < ApplicationController
    
      def show
        @user = User.find(params[:id])
      end
    
      def new
      end
    end

**Debugger**

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
    
      def show
        @user = User.find(params[:id])
        debugger
      end
    
      def new
      end
    end


**Gravatar e sidebar**

> app/views/users/show.html.erb

    <% provide(:title, @user.name) %>
    <h1>
      <%= gravatar_for @user %>
      <%= @user.name %>
    </h1>

> app/helpers/users_helper.rb

    module UsersHelper
    
      # Returns the Gravatar for the given user.
      def gravatar_for(user)
        gravatar_id = Digest::MD5::hexdigest(user.email.downcase)
        gravatar_url = "https://secure.gravatar.com/avatar/#{gravatar_id}"
        image_tag(gravatar_url, alt: user.name, class: "gravatar")
      end
    end


> app/views/users/show.html.erb

    <% provide(:title, @user.name) %>
    <div class="row">
      <aside class="col-md-4">
        <section class="user_info">
          <h1>
            <%= gravatar_for @user %>
            <%= @user.name %>
          </h1>
        </section>
      </aside>
    </div>

> app/assets/stylesheets/custom.css.scss

    .
    .
    .
    /* sidebar */
    
    aside {
      section.user_info {
        margin-top: 20px;
      }
      section {
        padding: 10px 0;
        margin-top: 20px;
        &:first-child {
          border: 0;
          padding-top: 0;
        }
        span {
          display: block;
          margin-bottom: 3px;
          line-height: 1;
        }
        h1 {
          font-size: 1.4em;
          text-align: left;
          letter-spacing: -1px;
          margin-bottom: 3px;
          margin-top: 0px;
        }
      }
    }
    
    .gravatar {
      float: left;
      margin-right: 10px;
    }
    
    .gravatar_edit {
      margin-top: 15px;
    }

**Formulário de registro de usuário**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_mockup_bootstrap.png)

    $ ./bin/rake db:migrate:reset

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
    
      def show
        @user = User.find(params[:id])
      end
    
      def new
        @user = User.new
      end
    end

> app/views/users/new.html.erb

    <% provide(:title, 'Sign up') %>
    <h1>Sign up</h1>
    
    <div class="row">
      <div class="col-md-6 col-md-offset-3">
        <%= form_for(@user) do |f| %>
          <%= f.label :name %>
          <%= f.text_field :name %>
    
          <%= f.label :email %>
          <%= f.email_field :email %>
    
          <%= f.label :password %>
          <%= f.password_field :password %>
    
          <%= f.label :password_confirmation, "Confirmation" %>
          <%= f.password_field :password_confirmation %>
    
          <%= f.submit "Create my account", class: "btn btn-primary" %>
        <% end %>
      </div>
    </div>

> app/assets/stylesheets/custom.css.scss

    .
    .
    .
    /* forms */
    
    input, textarea, select, .uneditable-input {
      border: 1px solid #bbb;
      width: 100%;
      margin-bottom: 15px;
      @include box_sizing;
    }
    
    input {
      height: auto !important;
    }


**Signups mal sucedidos**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_failure_mockup_bootstrap.png)

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
    
      def show
        @user = User.find(params[:id])
      end
    
      def new
        @user = User.new
      end
    
      def create
        @user = User.new(params[:user])    # Not the final implementation!
        if @user.save
          # Handle a successful save.
        else
          render 'new'
        end
      end
    end

**Strong parameters**

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
      .
      .
      .
      def create
        @user = User.new(user_params)
        if @user.save
          # Handle a successful save.
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

**Mensagens de erro no Signup**

    $ ./bin/rails console
    >> user = User.new(name: "Foo Bar", email: "foo@invalid",
    ?>                 password: "dude", password_confirmation: "dude")
    >> user.save
    => false
    >> user.errors.full_messages
    => ["Email is invalid", "Password is too short (minimum is 6 characters)"]

> app/views/users/new.html.erb

    <% provide(:title, 'Sign up') %>
    <h1>Sign up</h1>
    
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
    
          <%= f.submit "Create my account", class: "btn btn-primary" %>
        <% end %>
      </div>
    </div>

> app/views/shared/_error_messages.html.erb

    <% if @user.errors.any? %>
      <div id="error_explanation">
        <div class="alert alert-danger">
          The form contains <%= pluralize(@user.errors.count, "error") %>.
        </div>
        <ul>
        <% @user.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
        </ul>
      </div>
    <% end %>

> app/assets/stylesheets/custom.css.scss

    .
    .
    .
    /* forms */
    .
    .
    .
    #error_explanation {
      color: red;
      ul {
        color: red;
        margin: 0 0 30px 0;
      }
    }
    
    .field_with_errors {
      @extend .has-error;
      .form-control {
        color: $state-danger-text;
      }
    }


**Testando Signups inválidos**

    $ ./bin/rails generate integration_test users_signup

> test/integration/users_signup_test.rb

    require 'test_helper'
    
    class UsersSignupTest < ActionDispatch::IntegrationTest
    
      test "invalid signup information" do
        get signup_path
        assert_no_difference 'User.count' do
          post users_path, user: { name:  "",
                                   email: "user@invalid",
                                   password:              "foo",
                                   password_confirmation: "bar" }
        end
        assert_template 'users/new'
      end
    end

**Signups bem sucedidos**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_success_mockup_bootstrap.png)

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
      .
      .
      .
      def create
        @user = User.new(user_params)
        if @user.save
          redirect_to @user #redirect_to user_url(@user) é equivalente
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

Vamos mostrar uma mensagem depois do redirect usando flash:

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
      .
      .
      .
      def create
        @user = User.new(user_params)
        if @user.save
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

> app/views/layouts/application.html.erb

    <!DOCTYPE html>
    <html>
      .
      .
      .
      <body>
        <%= render 'layouts/header' %>
        <div class="container">
          <% flash.each do |message_type, message| %>
            <div class="alert alert-<%= message_type %>"><%= message %></div>
          <% end %>
          <%= yield %>
          <%= render 'layouts/footer' %>
          <%= debug(params) if Rails.env.development? %>
        </div>
        .
        .
        .
      </body>
    </html>
















