
**11 - Seguindo usuários**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_profile_mockup_3rd_edition.png)

o usuário calvin vai até a tela de listagem de usuários onde pode encontrar outro usuário para seguir:

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_user_index_mockup_bootstrap.png)

ele escolhe o usuário hobbes:

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_other_profile_follow_button_mockup_3rd_edition.png)

a contagem de followers do usuário hobbes aumenta e o botão "follow" vira "unfollow"

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_other_profile_unfollow_button_mockup_3rd_edition.png)

voltando para a homepage o usuário calvin agora também encontra listados os microposts do usuário hobbes

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_home_page_feed_mockup_3rd_edition.png)


**Proposta de modelo de relacionamento**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/naive_user_has_many_following.png)

problemas dessa proposta:

 - redundância de dados 
 - duas tabelas, com redundância de dados: followers e following
 - manter os dados de usuário atualizados seria muito difícil

**Relacionamento usando has_many through**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_has_many_following_3rd_edition.png)

usando arquitetura rest chegamos ao modelo relantionship:

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/relationship_model.png)

    $ ./bin/rails generate model Relationship follower_id:integer followed_id:integer

> db/migrate/[timestamp]_create_relationships.rb

    class CreateRelationships < ActiveRecord::Migration
      def change
        create_table :relationships do |t|
          t.integer :follower_id
          t.integer :followed_id
    
          t.timestamps null: false
        end
        add_index :relationships, :follower_id
        add_index :relationships, :followed_id
        add_index :relationships, [:follower_id, :followed_id], unique: true
      end
    end

**Associações de User/relationship**

> app/models/user.rb

    class User < ActiveRecord::Base
      has_many :microposts, dependent: :destroy
      has_many :active_relationships, class_name:  "Relationship",
                                      foreign_key: "follower_id",
                                      dependent:   :destroy
      .
      .
      .
    end

> app/models/relationship.rb

    class Relationship < ActiveRecord::Base
      belongs_to :follower, class_name: "User"
      belongs_to :followed, class_name: "User"
    end

com essas associações, os seguintes métodos passam a ficar disponíveis:

 - active_relationship.follower  => Retorna quem está seguindo(follower)
 - active_relationship.followed => Retorna quem está sendo seguido(followed)
 - user.active_relationships.create(followed_id: user.id) => Cria um relacionamento ativo com o usuário
 - user.active_relationships.create!(followed_id: user.id) => Cria um relacionamento ativo com o usuário (com exception em caso de falha)
 - user.active_relationships.build(followed_id: user.id) => Retorna um nov objeto Relationship associado ao usuário

**Validações de relacionamento**

> test/models/relationship_test.rb

    require 'test_helper'
    
    class RelationshipTest < ActiveSupport::TestCase
    
      def setup
        @relationship = Relationship.new(follower_id: 1, followed_id: 2)
      end
    
      test "should be valid" do
        assert @relationship.valid?
      end
    
      test "should require a follower_id" do
        @relationship.follower_id = nil
        assert_not @relationship.valid?
      end
    
      test "should require a followed_id" do
        @relationship.followed_id = nil
        assert_not @relationship.valid?
      end
    end

> app/models/relationship.rb

    class Relationship < ActiveRecord::Base
      belongs_to :follower, class_name: "User"
      belongs_to :followed, class_name: "User"
      validates :follower_id, presence: true
      validates :followed_id, presence: true
    end

> test/fixtures/relationships.yml

    # empty

**Followed users (usuários que estão sendo seguidos)**

> app/models/user.rb

    class User < ActiveRecord::Base
      has_many :microposts, dependent: :destroy
      has_many :active_relationships, class_name:  "Relationship",
                                      foreign_key: "follower_id",
                                      dependent:   :destroy
      has_many :following, through: :active_relationships, source: :followed
      .
      .
      .
    end

a implementação dessas associações nos permite, entre outras coisas, verificar se um usuário é seguidor de outro:

    user.following.include?(other_user)

ou buscar um usuário através do relacionamento:

    user.following.find(other_user)

Para manipular relationships vamos criar alguns métodos utilitários como follow e unfollow para que possamos, por exemplo, escrever user.follow(other_user). Um terceiro método, chamado following?, será um método booleano para testar se um usuário está seguindo outro. Mas antes vamos implementar os testes:

> test/models/user_test.rb

    require 'test_helper'
    
    class UserTest < ActiveSupport::TestCase
      .
      .
      .
      test "should follow and unfollow a user" do
        michael = users(:michael)
        archer  = users(:archer)
        assert_not michael.following?(archer)
        michael.follow(archer)
        assert michael.following?(archer)
        michael.unfollow(archer)
        assert_not michael.following?(archer)
      end
    end

> app/models/user.rb

    class User < ActiveRecord::Base
      .
      .
      .
      def feed
        .
        .
        .
      end
    
      # Follows a user.
      def follow(other_user)
        active_relationships.create(followed_id: other_user.id)
      end
    
      # Unfollows a user.
      def unfollow(other_user)
        active_relationships.find_by(followed_id: other_user.id).destroy
      end
    
      # Returns true if the current user is following the other user.
      def following?(other_user)
        following.include?(other_user)
      end
    
      private
      .
      .
      .
    end

**Followers (seguidores)**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_has_many_followers_3rd_edition.png)


> app/models/user.rb

    class User < ActiveRecord::Base
      has_many :microposts, dependent: :destroy
      has_many :active_relationships,  class_name:  "Relationship",
                                       foreign_key: "follower_id",
                                       dependent:   :destroy
      has_many :passive_relationships, class_name:  "Relationship",
                                       foreign_key: "followed_id",
                                       dependent:   :destroy
      has_many :following, through: :active_relationships,  source: :followed
      has_many :followers, through: :passive_relationships, source: :follower
      .
      .
      .
    end

> test/models/user_test.rb

    require 'test_helper'
    
    class UserTest < ActiveSupport::TestCase
      .
      .
      .
      test "should follow and unfollow a user" do
        michael  = users(:michael)
        archer   = users(:archer)
        assert_not michael.following?(archer)
        michael.follow(archer)
        assert michael.following?(archer)
        assert archer.followers.include?(michael)
        michael.unfollow(archer)
        assert_not michael.following?(archer)
      end
    end

**Interface web para seguir usuários**

Primeiro vamos adicionar relacionamentos de following/follower a nossas dados de exemplo: 

> db/seeds.rb
   
    .
    .
    .
    .
    # Following relationships
    users = User.all
    user  = users.first
    following = users[2..50]
    followers = users[3..40]
    following.each { |followed| user.follow(followed) }
    followers.each { |follower| follower.follow(user) }

para aplicar o código acima:

    $ bundle exec rake db:migrate:reset
    $ bundle exec rake db:seed

**Formulário de follow e número de seguidos e seguidores:**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/stats_partial_mockup.png)

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
      resources :users do
        member do
          get :following, :followers
        end
      end
      resources :microposts,          only: [:create, :destroy]
    end

com isso temos novas rotas disponíveis:

    HTTP request 	URL 	            Action 	Named route
    GET 	        /users/1/following 	following 	following_user_path(1)
    GET 	        /users/1/followers 	followers 	followers_user_path(1)


> app/views/shared/_stats.html.erb

    <% @user ||= current_user %>
    <div class="stats">
      <a href="<%= following_user_path(@user) %>">
        <strong id="following" class="stat">
          <%= @user.following.count %>
        </strong>
        following
      </a>
      <a href="<%= followers_user_path(@user) %>">
        <strong id="followers" class="stat">
          <%= @user.followers.count %>
        </strong>
        followers
      </a>
    </div>

> app/views/static_pages/home.html.erb

    <% if logged_in? %>
      <div class="row">
        <aside class="col-md-4">
          <section class="user_info">
            <%= render 'shared/user_info' %>
          </section>
          <section class="stats">
            <%= render 'shared/stats' %>
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

> app/assets/stylesheets/custom.css.scss

    .
    .
    .
    /* sidebar */
    .
    .
    .    
    .stats {
      overflow: auto;
      margin-top: 0;
      padding: 0;
      a {
        float: left;
        padding: 0 10px;
        border-left: 1px solid $gray-lighter;
        color: gray;
        &:first-child {
          padding-left: 0;
          border: 0;
        }
        &:hover {
          text-decoration: none;
          color: blue;
        }
      }
      strong {
        display: block;
      }
    }
    
    .user_avatars {
      overflow: auto;
      margin-top: 10px;
      .gravatar {
        margin: 1px 1px;
      }
      a {
        padding: 0;
      }
    }
    
    .users.follow {
      padding: 0;
    }
    
    /* forms */
    .
    .
    .


> app/views/users/_follow_form.html.erb

    <% unless current_user?(@user) %>
      <div id="follow_form">
      <% if current_user.following?(@user) %>
        <%= render 'unfollow' %>
      <% else %>
        <%= render 'follow' %>
      <% end %>
      </div>
    <% end %>

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
      resources :users do
        member do
          get :following, :followers
        end
      end
      resources :microposts,          only: [:create, :destroy]
      resources :relationships,       only: [:create, :destroy]
    end

> app/views/users/_follow.html.erb

    <%= form_for(current_user.active_relationships.build) do |f| %>
      <div><%= hidden_field_tag :followed_id, @user.id %></div>
      <%= f.submit "Follow", class: "btn btn-primary" %>
    <% end %>

> app/views/users/_unfollow.html.erb

    <%= form_for(current_user.active_relationships.find_by(followed_id: @user.id),
                 html: { method: :delete }) do |f| %>
      <%= f.submit "Unfollow", class: "btn" %>
    <% end %>

> app/views/users/show.html.erb

    <% provide(:title, @user.name) %>
    <div class="row">
      <aside class="col-md-4">
        <section>
          <h1>
            <%= gravatar_for @user %>
            <%= @user.name %>
          </h1>
        </section>
        <section class="stats">
          <%= render 'shared/stats' %>
        </section>
      </aside>
      <div class="col-md-8">
        <%= render 'follow_form' if logged_in? %>
        <% if @user.microposts.any? %>
          <h3>Microposts (<%= @user.microposts.count %>)</h3>
          <ol class="microposts">
            <%= render @microposts %>
          </ol>
          <%= will_paginate @microposts %>
        <% end %>
      </div>
    </div>

**Páginas de Following(seguidos) e followers(seguidores)**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/following_mockup_bootstrap.png)

> test/controllers/users_controller_test.rb

    require 'test_helper'
    
    class UsersControllerTest < ActionController::TestCase
    
      def setup
        @user = users(:michael)
        @other_user = users(:archer)
      end
      .
      .
      .
      test "should redirect following when not logged in" do
        get :following, id: @user
        assert_redirected_to login_url
      end
    
      test "should redirect followers when not logged in" do
        get :followers, id: @user
        assert_redirected_to login_url
      end
    end

> app/controllers/users_controller.rb

    class UsersController < ApplicationController
      before_action :logged_in_user, only: [:index, :edit, :update, :destroy,
                                            :following, :followers]
      .
      .
      .
      def following
        @title = "Following"
        @user  = User.find(params[:id])
        @users = @user.following.paginate(page: params[:page])
        render 'show_follow'
      end
    
      def followers
        @title = "Followers"
        @user  = User.find(params[:id])
        @users = @user.followers.paginate(page: params[:page])
        render 'show_follow'
      end
    
      private
      .
      .
      .
    end

> app/views/users/show_follow.html.erb

    <% provide(:title, @title) %>
    <div class="row">
      <aside class="col-md-4">
        <section class="user_info">
          <%= gravatar_for @user %>
          <h1><%= @user.name %></h1>
          <span><%= link_to "view my profile", @user %></span>
          <span><b>Microposts:</b> <%= @user.microposts.count %></span>
        </section>
        <section class="stats">
          <%= render 'shared/stats' %>
          <% if @users.any? %>
            <div class="user_avatars">
              <% @users.each do |user| %>
                <%= link_to gravatar_for(user, size: 30), user %>
              <% end %>
            </div>
          <% end %>
        </section>
      </aside>
      <div class="col-md-8">
        <h3><%= @title %></h3>
        <% if @users.any? %>
          <ul class="users follow">
            <%= render @users %>
          </ul>
          <%= will_paginate %>
        <% end %>
      </div>
    </div>
























