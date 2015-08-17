
6 - User model
---------------


![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_model_sketch.png)

$ ./bin/rails generate model User name:string email:string

> db/migrate/[timestamp]_create_users.rb

    class CreateUsers < ActiveRecord::Migration
      def change
        create_table :users do |t|
          t.string :name
          t.string :email
    
          t.timestamps null: false
        end
      end
    end

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_model_initial_3rd_edition.png)

    $ ./bin/rake db:migrate

**DB Browser for SQLite**

    $ sudo apt-get install sqlitebrowser
    
   para desfazer alguma mudança no banco de dados:

    $ ./bin/rake db:rollback

**Criando objetos User**

    $ ./bin/rails console --sandbox

    >> User.new
    => #<User id: nil, name: nil, email: nil, created_at: nil, updated_at: nil>
    >> user = User.new(name: "Lucas Pratto", email: "pratto@gnv.com")
    => #<User id: nil, name: "Lucas Pratto", email: "pratto@gnv.com", created_at: nil, updated_at: nil>
    >> user.valid?
    true
    >> user.save
    => true
    >> user
    >> User.create(name: "Diego Tardelli", email: "tardelli@gnv.com")
    >> walking_dead = User.create(name: "Walking dead", email: "keepwalking@dead.com")
    >> walking_dead.destroy
    >> walking_dead

**Buscando objetos user**

    >> User.find(1)
    >> User.find(3)
    ActiveRecord::RecordNotFound: Couldn't find User with ID=3
    >> User.find_by(email: "pratto@gnv.com")
    >> User.first
    >> User.all

**Editando objetos user**

    >> user.email = "pratto@gnv.net"
    => "pratto@gnv.net"
    >> user.save
    => true
    >> user.email
    => "pratto@gnv.net"
    >> user.email = "foo@bar.com"
    => "foo@bar.com"
    >> user.reload.email
    => "pratto@gnv.net"
    >> user.created_at
    => "2015-07-24 00:57:46"
    >> user.updated_at
    => "2015-07-24 01:37:32"
    >> user.update_attributes(name: "The Dude", email: "dude@abides.org")
    => true
    >> user.name
    => "The Dude"
    >> user.email
    => "dude@abides.org"

**Testes de validação**

> test/models/user_test.rb

    require 'test_helper'
    
    class UserTest < ActiveSupport::TestCase
    
      def setup
        @user = User.new(name: "Example User", email: "user@example.com")
      end
    
      test "should be valid" do
        assert @user.valid?
      end
    end

**Validando presença**

> test/models/user_test.rb

    require 'test_helper'

    class UserTest < ActiveSupport::TestCase
    
      def setup
        @user = User.new(name: "Example User", email: "user@example.com")
      end
    
      test "should be valid" do
        assert @user.valid?
      end
    
      test "name should be present" do
        @user.name = "     "
        assert_not @user.valid?
      end
    end

> app/models/user.rb

    class User < ActiveRecord::Base
      validates :name, presence: true
    end

> test/models/user_test.rb

    require 'test_helper'
    
    class UserTest < ActiveSupport::TestCase
    
      def setup
        @user = User.new(name: "Example User", email: "user@example.com")
      end
    
      test "should be valid" do
        assert @user.valid?
      end
    
      test "name should be present" do
        @user.name = ""
        assert_not @user.valid?
      end
    
      test "email should be present" do
        @user.email = "     "
        assert_not @user.valid?
      end
    end

> app/models/user.rb

    class User < ActiveRecord::Base
      validates :name,  presence: true
      validates :email, presence: true
    end








