
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









