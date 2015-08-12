
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
        assert_select "title", "Ruby on Rails Tutorial Sample App"
      end
    
      test "should get help" do
        get :help
        assert_response :success
        assert_select "title", "Help | Ruby on Rails Tutorial Sample App"
      end
    
      test "should get about" do
        get :about
        assert_response :success
        assert_select "title", "About | Ruby on Rails Tutorial Sample App"
      end
    end


