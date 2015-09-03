
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





