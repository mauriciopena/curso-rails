
**10 - Microposts**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/micropost_model_3rd_edition.png)

    $ ./bin/rails generate model Micropost content:text user:references

> db/migrate/[timestamp]_create_microposts.rb

    class CreateMicroposts < ActiveRecord::Migration
      def change
        create_table :microposts do |t|
          t.text :content
          t.references :user, index: true, foreign_key: true
    
          t.timestamps null: false
        end
        add_index :microposts, [:user_id, :created_at]
      end
    end


**Validações de Micropost**

> test/models/micropost_test.rb

    require 'test_helper'
    
    class MicropostTest < ActiveSupport::TestCase
    
      def setup
        @user = users(:michael)
        # This code is not idiomatically correct.
        @micropost = Micropost.new(content: "Lorem ipsum", user_id: @user.id)
      end
    
      test "should be valid" do
        assert @micropost.valid?
      end
    
      test "user id should be present" do
        @micropost.user_id = nil
        assert_not @micropost.valid?
      end
    end

> app/models/micropost.rb

    class Micropost < ActiveRecord::Base
       belongs_to :user
       validates :user_id, presence: true
    end

> test/models/micropost_test.rb

      .
      .
      .
      test "content should be present" do
        @micropost.content = "   "
        assert_not @micropost.valid?
      end
    
      test "content should be at most 140 characters" do
        @micropost.content = "a" * 141
        assert_not @micropost.valid?
      end
    end

> app/models/micropost.rb

    class Micropost < ActiveRecord::Base
      belongs_to :user
      validates :user_id, presence: true
      validates :content, presence: true, length: { maximum: 140 }
    end


**Associação de User/Micropost**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/micropost_belongs_to_user.png)

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_has_many_microposts.png)

Usando as associações belongs_to/has_many o Rails nos disponibiliza os seguintes métodos:

 - micropost.user : Retorna o objeto User associado ao micropost
 - user.microposts : Returna um hash com os microposts associados ao objeto user
 - user.microposts.create(arg) : Grava um novo micropost associado ao objeto user
 - user.microposts.create!(arg) : Grava um novo micropost associado ao objeto user (lança uma exceção em caso de falha)
 - user.microposts.build(arg) : Returna um novo objeto Micropost associado ao user
 - user.microposts.find_by(id: 1) : Busca o micropost com id igual a 1 e user_id igual a user.id

> app/models/user.rb

    class User < ActiveRecord::Base
      has_many :microposts
      .
      .
      .
    end

> test/models/micropost_test.rb

    require 'test_helper'
    
    class MicropostTest < ActiveSupport::TestCase
    
      def setup
        @user = users(:michael)
        @micropost = @user.microposts.build(content: "Lorem ipsum")
      end
    
      test "should be valid" do
        assert @micropost.valid?
      end
    
      test "user id should be present" do
        @micropost.user_id = nil
        assert_not @micropost.valid?
      end
      .
      .
      .
    end

**Melhoramentos no Micropost**

primeiro vamos usar o default_scope do rails para ordenar os microposts de um usuário para que venham na ordem reversa da postagem de maneira que os mais novos apareçam primeiro.

> test/models/micropost_test.rb

    require 'test_helper'
    
    class MicropostTest < ActiveSupport::TestCase
      .
      .
      .
      test "order should be most recent first" do
        assert_equal microposts(:most_recent), Micropost.first
      end
    end

> test/fixtures/microposts.yml

    orange:
      content: "I just ate an orange!"
      created_at: <%= 10.minutes.ago %>
    
    tau_manifesto:
      content: "Check out the @tauday site by @mhartl: http://tauday.com"
      created_at: <%= 3.years.ago %>
    
    cat_video:
      content: "Sad cats are sad: http://youtu.be/PKffm2uI4dk"
      created_at: <%= 2.hours.ago %>
    
    most_recent:
      content: "Writing a short test"
      created_at: <%= Time.zone.now %>

> app/models/micropost.rb

    class Micropost < ActiveRecord::Base
      belongs_to :user
      default_scope -> { order(:created_at => :desc) }
      validates :user_id, presence: true
      validates :content, presence: true, length: { maximum: 140 }
    end

**Dependent: destroy**

> test/models/user_test.rb

    require 'test_helper'
    
    class UserTest < ActiveSupport::TestCase
    
      def setup
        @user = User.new(name: "Example User", email: "user@example.com",
                         password: "foobar", password_confirmation: "foobar")
      end
      .
      .
      .
      test "associated microposts should be destroyed" do
        @user.save
        @user.microposts.create!(content: "Lorem ipsum")
        assert_difference 'Micropost.count', -1 do
          @user.destroy
        end
      end
    end

> app/models/user.rb

    class User < ActiveRecord::Base
      has_many :microposts, :dependent => :destroy
      .
      .
      .
    end

**Listando os microposts**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_microposts_mockup_3rd_edition.png)

    $ ./bin/rails generate controller Microposts

> app/views/microposts/_micropost.html.erb

    <li id="micropost-<%= micropost.id %>">
      <%= link_to gravatar_for(micropost.user, size: 50), micropost.user %>
      <span class="user"><%= link_to micropost.user.name, micropost.user %></span>
      <span class="content"><%= micropost.content %></span>
      <span class="timestamp">
        Posted <%= time_ago_in_words(micropost.created_at) %> ago.
      </span>
    </li>

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
      .
      .
      .
      def show
        @user = User.find(params[:id])
        @microposts = @user.microposts.paginate(page: params[:page])
      end
      .
      .
      .
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
      <div class="col-md-8">
        <% if @user.microposts.any? %>
          <h3>Microposts (<%= @user.microposts.count %>)</h3>
          <ol class="microposts">
            <%= render @microposts %>
          </ol>
          <%= will_paginate @microposts %>
        <% end %>
      </div>
    </div>


**Polulando o BD com Microposts**

> db/seeds.rb

    .
    .
    .
    users = User.order(:created_at).take(6)
    50.times do
      content = Faker::Lorem.sentence(5)
      users.each { |user| user.microposts.create!(content: content) }
    end

agora podemos reiniciar e popular o BD:

    $ bundle exec rake db:migrate:reset
    $ bundle exec rake db:seed

> app/assets/stylesheets/custom.css.scss

    .
    .
    .
    /* microposts */
    
    .microposts {
      list-style: none;
      padding: 0;
      li {
        padding: 10px 0;
        border-top: 1px solid #e8e8e8;
      }
      .user {
        margin-top: 5em;
        padding-top: 0;
      }
      .content {
        display: block;
        margin-left: 60px;
        img {
          display: block;
          padding: 5px 0;
        }
      }
      .timestamp {
        color: $gray-light;
        display: block;
        margin-left: 60px;
      }
      .gravatar {
        float: left;
        margin-right: 10px;
        margin-top: 5px;
      }
    }
    
    aside {
      textarea {
        height: 100px;
        margin-bottom: 5px;
      }
    }
    
    span.picture {
      margin-top: 10px;
      input {
        border: 0;
      }
    }

**Testando a página de profile do usuário (e listagem de microposts)**

    $ ./bin/rails generate integration_test users_profile

> test/fixtures/microposts.yml

    orange:
      content: "I just ate an orange!"
      created_at: <%= 10.minutes.ago %>
      user: michael
    
    tau_manifesto:
      content: "Check out the @tauday site by @mhartl: http://tauday.com"
      created_at: <%= 3.years.ago %>
      user: michael
    
    cat_video:
      content: "Sad cats are sad: http://youtu.be/PKffm2uI4dk"
      created_at: <%= 2.hours.ago %>
      user: michael
    
    most_recent:
      content: "Writing a short test"
      created_at: <%= Time.zone.now %>
      user: michael
    
    <% 30.times do |n| %>
    micropost_<%= n %>:
      content: <%= Faker::Lorem.sentence(5) %>
      created_at: <%= 42.days.ago %>
      user: michael
    <% end %>

> test/integration/users_profile_test.rb

    require 'test_helper'
    
    class UsersProfileTest < ActionDispatch::IntegrationTest
      include ApplicationHelper
    
      def setup
        @user = users(:michael)
      end
    
      test "profile display" do
        get user_path(@user)
        assert_template 'users/show'
        assert_select 'title', full_title(@user.name)
        assert_select 'h1', text: @user.name
        assert_select 'h1>img.gravatar'
        assert_match @user.microposts.count.to_s, response.body
        assert_select 'div.pagination'
        @user.microposts.paginate(page: 1).each do |micropost|
          assert_match micropost.content, response.body
        end
      end
    end

**Manipulando Microposts**

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
      resources :microposts,          only: [:create, :destroy]
    end

**testes para testar a autorização nos microposts**

> test/controllers/microposts_controller_test.rb

    require 'test_helper'
    
    class MicropostsControllerTest < ActionController::TestCase
    
      def setup
        @micropost = microposts(:orange)
      end
    
      test "should redirect create when not logged in" do
        assert_no_difference 'Micropost.count' do
          post :create, micropost: { content: "Lorem ipsum" }
        end
        assert_redirected_to login_url
      end
    
      test "should redirect destroy when not logged in" do
        assert_no_difference 'Micropost.count' do
          delete :destroy, id: @micropost
        end
        assert_redirected_to login_url
      end
    end

para evitar duplicação de código, vamos mover o método logged_in_user do users_controller para o application_controller. Assim ele ficará disponível tanto no users_controller quanto no microposts_controller:

> app/controllers/application_controller.rb

    class ApplicationController < ActionController::Base
      protect_from_forgery with: :exception
      include SessionsHelper
    
      private
    
        # Confirms a logged-in user.
        def logged_in_user
          unless logged_in?
            store_location
            flash[:danger] = "Please log in."
            redirect_to login_url
          end
        end
    end


> app/controllers/microposts_controller.rb

    class MicropostsController < ApplicationController
      before_action :logged_in_user, only: [:create, :destroy]
    
      def create
      end
    
      def destroy
      end
    end

**Cadastrando microposts**

> app/controllers/microposts_controller.rb

    class MicropostsController < ApplicationController
      before_action :logged_in_user, only: [:create, :destroy]
    
      def create
        @micropost = current_user.microposts.build(micropost_params)
        if @micropost.save
          flash[:success] = "Micropost created!"
          redirect_to root_url
        else
          render 'static_pages/home'
        end
      end
    
      def destroy
      end
    
      private
    
        def micropost_params
          params.require(:micropost).permit(:content)
        end
    end

> app/views/static_pages/home.html.erb

    <% if logged_in? %>
      <div class="row">
        <aside class="col-md-4">
          <section class="user_info">
            <%= render 'shared/user_info' %>
          </section>
          <section class="micropost_form">
            <%= render 'shared/micropost_form' %>
          </section>
        </aside>
      </div>
    <% else %>
      <div class="center jumbotron">
        <h1>Welcome to the Sample App</h1>
    
        <h2>
          This is the home page.
        </h2>
    
        <%= link_to "Sign up now!", signup_path, class: "btn btn-lg btn-primary" %>
      </div>
    
      <%= link_to image_tag("rails.png", alt: "Rails logo"),
                  'http://rubyonrails.org/' %>
    <% end %>

> app/views/shared/_user_info.html.erb

    <%= link_to gravatar_for(current_user, size: 50), current_user %>
    <h1><%= current_user.name %></h1>
    <span><%= link_to "view my profile", current_user %></span>
    <span><%= pluralize(current_user.microposts.count, "micropost") %></span>

> app/views/shared/_micropost_form.html.erb

    <%= form_for(@micropost) do |f| %>
      <%= render 'shared/error_messages', object: f.object %>
      <div class="field">
        <%= f.text_area :content, placeholder: "Compose new micropost..." %>
      </div>
      <%= f.submit "Post", class: "btn btn-primary" %>
    <% end %>

> app/controllers/static_pages_controller.rb

    class StaticPagesController < ApplicationController
    
      def home
        @micropost = current_user.microposts.build if logged_in?
      end
    
      def help
      end
    
      def about
      end
    
      def contact
      end
    end

> app/views/shared/_error_messages.html.erb

    <% if object.errors.any? %>
      <div id="error_explanation">
        <div class="alert alert-danger">
          The form contains <%= pluralize(object.errors.count, "error") %>.
        </div>
        <ul>
        <% object.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
        </ul>
      </div>
    <% end %>

> app/views/users/new.html.erb

    <% provide(:title, 'Sign up') %>
    <h1>Sign up</h1>
    
    <div class="row">
      <div class="col-md-6 col-md-offset-3">
        <%= form_for(@user) do |f| %>
          <%= render 'shared/error_messages', object: f.object %>
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

> app/views/users/edit.html.erb

     <% provide(:title, "Edit user") %>
    <h1>Update your profile</h1>
    
    <div class="row">
      <div class="col-md-6 col-md-offset-3">
        <%= form_for(@user) do |f| %>
          <%= render 'shared/error_messages', object: f.object %>
    
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
          <a href="http://gravatar.com/emails">change</a>
        </div>
      </div>
    </div>

**Feed de microposts**

mock da home page com feed:

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/proto_feed_mockup_3rd_edition.png)

> app/models/user.rb

    class User < ActiveRecord::Base
      .
      .
      .
      # Defines a proto-feed.      
      def feed
        Micropost.where("user_id = ?", id)
      end
    
        private
        .
        .
        .
    end

> app/controllers/static_pages_controller.rb

    class StaticPagesController < ApplicationController
    
      def home
        if logged_in?
          @micropost  = current_user.microposts.build
          @feed_items = current_user.feed.paginate(page: params[:page])
        end
      end
    
      def help
      end
    
      def about
      end
    
      def contact
      end
    end

> app/views/shared/_feed.html.erb

    <% if @feed_items.any? %>
      <ol class="microposts">
        <%= render @feed_items %>
      </ol>
      <%= will_paginate @feed_items %>
    <% end %>

> app/views/static_pages/home.html.erb

    <% if logged_in? %>
      <div class="row">
        <aside class="col-md-4">
          <section class="user_info">
            <%= render 'shared/user_info' %>
          </section>
          <section class="micropost_form">
            <%= render 'shared/micropost_form' %>
          </section>
        </aside>
        <div class="col-md-8">
          <h3>Micropost Feed</h3>
          <%= render 'shared/feed' %>
        </div>
      </div>
    <% else %>
      .
      .
      .
    <% end %>

> app/controllers/microposts_controller.rb

    class MicropostsController < ApplicationController
      before_action :logged_in_user, only: [:create, :destroy]
    
      def create
        @micropost = current_user.microposts.build(micropost_params)
        if @micropost.save
          flash[:success] = "Micropost created!"
          redirect_to root_url
        else
          @feed_items = []
          render 'static_pages/home'
        end
      end
    
      def destroy
      end
    
      private
    
        def micropost_params
          params.require(:micropost).permit(:content)
        end
    end

> app/controllers/microposts_controller.rb

    class MicropostsController < ApplicationController
      .
      .
      .
      def create
        @micropost = current_user.microposts.build(micropost_params)
        if @micropost.save
          flash[:success] = "Micropost created!"
          redirect_to root_url
        else
          @feed_items = []
          render 'static_pages/home'
        end
      end
      .
      .
      .
    end

**Exlcuíndo microposts**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/micropost_delete_links_mockup_3rd_edition.png)

> app/views/microposts/_micropost.html.erb

    <li id="<%= micropost.id %>">
      <%= link_to gravatar_for(micropost.user, size: 50), micropost.user %>
      <span class="user"><%= link_to micropost.user.name, micropost.user %></span>
      <span class="content"><%= micropost.content %></span>
      <span class="timestamp">
        Posted <%= time_ago_in_words(micropost.created_at) %> ago.
        <% if current_user?(micropost.user) %>
          <%= link_to "delete", micropost, method: :delete,
                                           data: { confirm: "You sure?" } %>
        <% end %>
      </span>
    </li>

> app/controllers/microposts_controller.rb

    class MicropostsController < ApplicationController
      before_action :logged_in_user, only: [:create, :destroy]
      before_action :correct_user,   only: :destroy
      .
      .
      .
      def destroy
        @micropost.destroy
        flash[:success] = "Micropost deleted"
        redirect_to request.referrer || root_url
      end
    
      private
    
        def micropost_params
          params.require(:micropost).permit(:content)
        end
    
         def correct_user
          @micropost = current_user.microposts.find_by(id: params[:id])
          redirect_to root_url if @micropost.nil?
        end
    end

**Testes de Micropost**

> test/fixtures/microposts.yml

    .
    .
    .
    ants:
      content: "Oh, is that what you want? Because that's how you get ants!"
      created_at: <%= 2.years.ago %>
      user: archer
    
    zone:
      content: "Danger zone!"
      created_at: <%= 3.days.ago %>
      user: archer
    
    tone:
      content: "I'm sorry. Your words made sense, but your sarcastic tone did not."
      created_at: <%= 10.minutes.ago %>
      user: lana
    
    van:
      content: "Dude, this van's, like, rolling probable cause."
      created_at: <%= 4.hours.ago %>
      user: lana

> test/controllers/microposts_controller_test.rb
   
      .
      .
      .
      test "should redirect destroy for wrong micropost" do
        log_in_as(users(:michael))
        micropost = microposts(:ants)
        assert_no_difference 'Micropost.count' do
          delete :destroy, id: micropost
        end
        assert_redirected_to root_url
      end
    end

Para finalizar vamos implementar um teste de integração em que fazemos login, verficamos a paginação dos microposts, tentamos cadastrar enviando dados inválidos, cadastramos com dados válidos, excluímos um micropost e depois visitamos a página de um outro usuário para verficar que os links de delete não aparecem.

    $ ./bin/rails generate integration_test microposts_interface

> test/integration/microposts_interface_test.rb

    require 'test_helper'
    
    class MicropostsInterfaceTest < ActionDispatch::IntegrationTest
    
      def setup
        @user = users(:michael)
      end
    
      test "micropost interface" do
        log_in_as(@user)
        get root_path
        assert_select 'div.pagination'
        # Invalid submission
        assert_no_difference 'Micropost.count' do
          post microposts_path, micropost: { content: "" }
        end
        assert_select 'div#error_explanation'
        # Valid submission
        content = "This micropost really ties the room together"
        assert_difference 'Micropost.count', 1 do
          post microposts_path, micropost: { content: content }
        end
        assert_redirected_to root_url
        follow_redirect!
        assert_match content, response.body
        # Delete a post.
        assert_select 'a', text: 'delete'
        first_micropost = @user.microposts.paginate(page: 1).first
        assert_difference 'Micropost.count', -1 do
          delete micropost_path(first_micropost)
        end
        # Visit a different user.
        get user_path(users(:archer))
        assert_select 'a', text: 'delete', count: 0
      end
    end

**Imagens em microposts**

> Gemfile

    .
    .
    .
    gem 'carrierwave'
    gem 'mini_magick'


A gem carrierwave tem um gerador que podemos usar para criar nosso uploader:

    $ ./bin/rails generate uploader Picture

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/micropost_model_picture.png)

    $ ./bin/rails generate migration add_picture_to_microposts picture:string
    $ ./bin/rake db:migrate

> app/models/micropost.rb

    class Micropost < ActiveRecord::Base
      belongs_to :user
      default_scope -> { order(created_at: :desc) }
      mount_uploader :picture, PictureUploader
      validates :user_id, presence: true
      validates :content, presence: true, length: { maximum: 140 }
    end

> app/views/shared/_micropost_form.html.erb

    <%= form_for(@micropost, html: { multipart: true }) do |f| %>
      <%= render 'shared/error_messages', object: f.object %>
      <div class="field">
        <%= f.text_area :content, placeholder: "Compose new micropost..." %>
      </div>
      <%= f.submit "Post", class: "btn btn-primary" %>
      <span class="picture">
        <%= f.file_field :picture %>
      </span>
    <% end %>

> app/controllers/microposts_controller.rb

    class MicropostsController < ApplicationController
      before_action :logged_in_user, only: [:create, :destroy]
      before_action :correct_user,   only: :destroy
      .
      .
      .
      private
    
        def micropost_params
          params.require(:micropost).permit(:content, :picture)
        end
    
         def correct_user
          @micropost = current_user.microposts.find_by(id: params[:id])
          redirect_to root_url if @micropost.nil?
        end
    end

> app/views/microposts/_micropost.html.erb

    <li id="micropost-<%= micropost.id %>">
      <%= link_to gravatar_for(micropost.user, size: 50), micropost.user %>
      <span class="user"><%= link_to micropost.user.name, micropost.user %></span>
      <span class="content">
        <%= micropost.content %>
        <%= image_tag micropost.picture.url if micropost.picture? %>
      </span>
      <span class="timestamp">
        Posted <%= time_ago_in_words(micropost.created_at) %> ago.
        <% if current_user?(micropost.user) %>
          <%= link_to "delete", micropost, method: :delete,
                                           data: { confirm: "You sure?" } %>
        <% end %>
      </span>
    </li>

**Validações de imagem**


    












































