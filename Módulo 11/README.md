
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

    $ ./bin/rails generate integration_test following

> test/fixtures/relationships.yml

    one:
      follower: michael
      followed: lana
    
    two:
      follower: michael
      followed: mallory
    
    three:
      follower: lana
      followed: michael
    
    four:
      follower: archer
      followed: michael

> test/integration/following_test.rb

    require 'test_helper'
    
    class FollowingTest < ActionDispatch::IntegrationTest
    
      def setup
        @user = users(:michael)
        log_in_as(@user)
      end
    
      test "following page" do
        get following_user_path(@user)
        assert_not @user.following.empty?
        assert_match @user.following.count.to_s, response.body
        @user.following.each do |user|
          assert_select "a[href=?]", user_path(user)
        end
      end
    
      test "followers page" do
        get followers_user_path(@user)
        assert_not @user.followers.empty?
        assert_match @user.followers.count.to_s, response.body
        @user.followers.each do |user|
          assert_select "a[href=?]", user_path(user)
        end
      end
    end

**Implementação do botão "follow"**

    $ ./bin/rails generate controller Relationships

> test/controllers/relationships_controller_test.rb

    require 'test_helper'
    
    class RelationshipsControllerTest < ActionController::TestCase
    
      test "create should require logged-in user" do
        assert_no_difference 'Relationship.count' do
          post :create
        end
        assert_redirected_to login_url
      end
    
      test "destroy should require logged-in user" do
        assert_no_difference 'Relationship.count' do
          delete :destroy, id: relationships(:one)
        end
        assert_redirected_to login_url
      end
    end

> app/controllers/relationships_controller.rb

    class RelationshipsController < ApplicationController
      before_action :logged_in_user
    
      def create
      end
    
      def destroy
      end
    end


> app/controllers/relationships_controller.rb

    class RelationshipsController < ApplicationController
      before_action :logged_in_user
    
      def create
        user = User.find(params[:followed_id])
        current_user.follow(user)
        redirect_to user
      end
    
      def destroy
        user = Relationship.find(params[:id]).followed
        current_user.unfollow(user)
        redirect_to user
      end
    end

**Botão follow com ajax**

> app/views/users/_follow.html.erb

    <%= form_for(current_user.active_relationships.build, remote: true) do |f| %>
      <div><%= hidden_field_tag :followed_id, @user.id %></div>
      <%= f.submit "Follow", class: "btn btn-primary" %>
    <% end %>

> app/views/users/_unfollow.html.erb

    <%= form_for(current_user.active_relationships.find_by(followed_id: @user.id),
                 html: { method: :delete },
                 remote: true) do |f| %>
      <%= f.submit "Unfollow", class: "btn" %>
    <% end %>

> app/controllers/relationships_controller.rb

    class RelationshipsController < ApplicationController
      before_action :logged_in_user
    
      def create
        @user = User.find(params[:followed_id])
        current_user.follow(@user)
        respond_to do |format|
          format.html { redirect_to @user }
          format.js
        end
      end
    
      def destroy
        @user = Relationship.find(params[:id]).followed
        current_user.unfollow(@user)
        respond_to do |format|
          format.html { redirect_to @user }
          format.js
        end
      end
    end

Para que nossos botões continuem funcionando mesmo que o browser não tenha javascript habilitado precisamos fazer uma pequena alteração de configuração:

> config/application.rb

    require File.expand_path('../boot', __FILE__)
    .
    .
    .
    module SampleApp
      class Application < Rails::Application
        .
        .
        .
        # Include the authenticity token in remote forms.
        config.action_view.embed_authenticity_token_in_remote_forms = true
      end
    end

> app/views/relationships/create.js.erb

    $("#follow_form").html("<%= escape_javascript(render('users/unfollow')) %>");
    $("#followers").html('<%= @user.followers.count %>');

> app/views/relationships/destroy.js.erb

    $("#follow_form").html("<%= escape_javascript(render('users/follow')) %>");
    $("#followers").html('<%= @user.followers.count %>');

**Testes de Following**

> test/integration/following_test.rb

    require 'test_helper'
    
    class FollowingTest < ActionDispatch::IntegrationTest
    
      def setup
        @user  = users(:michael)
        @other = users(:archer)
        log_in_as(@user)
      end
      .
      .
      .
      test "should follow a user the standard way" do
        assert_difference '@user.following.count', 1 do
          post relationships_path, followed_id: @other.id
        end
      end
    
      test "should follow a user with Ajax" do
        assert_difference '@user.following.count', 1 do
          xhr :post, relationships_path, followed_id: @other.id
        end
      end
    
      test "should unfollow a user the standard way" do
        @user.follow(@other)
        relationship = @user.active_relationships.find_by(followed_id: @other.id)
        assert_difference '@user.following.count', -1 do
          delete relationship_path(relationship)
        end
      end
    
      test "should unfollow a user with Ajax" do
        @user.follow(@other)
        relationship = @user.active_relationships.find_by(followed_id: @other.id)
        assert_difference '@user.following.count', -1 do
          xhr :delete, relationship_path(relationship)
        end
      end
    end

**O status feed**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_home_page_feed_mockup_bootstrap.png)

O feed de um usuário (id 1) que está seguindo os usuários com ids 2, 7, 8, e 10 ficaria assim:

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_feed.png)


> test/models/user_test.rb

    require 'test_helper'
    
    class UserTest < ActiveSupport::TestCase
      .
      .
      .
      test "feed should have the right posts" do
        michael = users(:michael)
        archer  = users(:archer)
        lana    = users(:lana)
        # Posts from followed user
        lana.microposts.each do |post_following|
          assert michael.feed.include?(post_following)
        end
        # Posts from self
        michael.microposts.each do |post_self|
          assert michael.feed.include?(post_self)
        end
        # Posts from unfollowed user
        archer.microposts.each do |post_unfollowed|
          assert_not michael.feed.include?(post_unfollowed)
        end
      end
    end

**Uma primeira implementação do feed**

A idéia é contruir uma query parecida com isso:

    SELECT * FROM microposts
    WHERE user_id IN (<list of ids>) OR user_id = <user id>

Uma maneira de fazer isso seria usando o método map que converte um array de inteiros em um array de strings:

    $ rails console
    >> [1, 2, 3, 4].map { |i| i.to_s }
    => ["1", "2", "3", "4"]

Situações como essa, em que o mesmo método é chamado em cada elemento de uma coleção,  são tão comuns que existe uma sintaxe especial que usa um "&" e um símbolo correspondente ao método:

    >> [1, 2, 3, 4].map(&:to_s)
    => ["1", "2", "3", "4"]

Usando o método join podemos criar uma string separada por vírgulas:

    >> [1, 2, 3, 4].map(&:to_s).join(', ')
    => "1, 2, 3, 4"

Podemos, então, usar o método acima para contruir o array de ids necessário assim:

    >> User.first.following.map(&:id)
    => [4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23,
    24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42,
    43, 44, 45, 46, 47, 48, 49, 50, 51]

Na verdade, que essa contrução é tão útil que ela já existe no Active Record:

    >> User.first.following_ids
    => [4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23,
    24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42,
    43, 44, 45, 46, 47, 48, 49, 50, 51]
    
    >> User.first.following_ids.join(', ')
    => "4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19,
    20, 21, 22, 23,24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34,35,
    36, 37, 38, 39, 40, 41, 42,43, 44, 45, 46, 47, 48, 49, 50, 51"








































