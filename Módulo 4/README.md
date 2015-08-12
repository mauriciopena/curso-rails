
4 - Ruby
--------

> app/helpers/application_helper.rb

    module ApplicationHelper    
      # Returns the full title on a per-page basis.
      def full_title(page_title = '')
        base_title = "Curso de Ruby on Rails"
        if page_title.empty?
          base_title
        else
          page_title + " | " + base_title
        end
      end
    end

> app/views/layouts/application.html.erb

    <!DOCTYPE html>
    <html>
      <head>
        <title><%= full_title(yield(:title)) %></title>
        <%= stylesheet_link_tag    'application', media: 'all',
                                                  'data-turbolinks-track' => true %>
        <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
        <%= csrf_meta_tags %>
      </head>
      <body>
        <%= yield %>
      </body>
    </html>

> test/controllers/static_pages_controller_test.rb

    require 'test_helper'
    
    class StaticPagesControllerTest < ActionController::TestCase
      test "should get home" do
        get :home
        assert_response :success
        assert_select "title", "#{@base_title}"
      end
    
      test "should get help" do
        get :help
        assert_response :success
        assert_select "title", "Help | #{@base_title}"
      end
    
      test "should get about" do
        get :about
        assert_response :success
        assert_select "title", "About | #{@base_title}"
      end
    end

> app/views/static_pages/home.html.erb

    <h1>Sample App</h1>
    <p>
      This is the home page for the
      <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
      sample application.
    </p>


** 4.1 Strings e methods**


----------


**4.1.1 strings**

    $ ./bin/rails c
    >> ""         # Uma string vazia
    => ""
    >> "foo"      # Uma string não vazia
    => "foo"

para concatenar string usamos o operador +:

    >> "foo" + "bar"    # String concatenation
    => "foobar"

Outro maneira é usando a sintaxe especial #{}:

>> first_name = "Lucas"    # Definindo variável
=> "Lucas"
>> "#{first_name} Pratto"     # String interpolation
=> "Lucas Pratto"

Outra maneira (menos elegante) de fazer a mesma coisa:

>> first_name = "Lucas"
=> "Lucas"
>> last_name = "Pratto"
=> "Pratto"
>> first_name + " " + last_name    # Concatenation, with a space in between
=> "Lucas Pratto"
>> "#{first_name} #{last_name}"    # The equivalent interpolation
=> "Lucas Pratto"





