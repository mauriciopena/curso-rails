

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













