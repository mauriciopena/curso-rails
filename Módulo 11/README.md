
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


















