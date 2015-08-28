
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
















