

5 -  Preenchendo o layout
----------------------

> app/views/layouts/application.html.erb

    <!DOCTYPE html>
    <html>
      <head>
        <title><%= full_title(yield(:title)) %></title>
        <%= stylesheet_link_tag 'application', media: 'all',
                                               'data-turbolinks-track' => true %>
        <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
        <%= csrf_meta_tags %>
        <!--[if lt IE 9]>
          <script src="//cdnjs.cloudflare.com/ajax/libs/html5shiv/r29/html5.min.js">
          </script>
        <![endif]-->
      </head>
      <body>
        <header class="navbar navbar-fixed-top navbar-inverse">
          <div class="container">
            <%= link_to "sample app", '#', id: "logo" %>
            <nav>
              <ul class="nav navbar-nav navbar-right">
                <li><%= link_to "Home",   '#' %></li>
                <li><%= link_to "Help",   '#' %></li>
                <li><%= link_to "Log in", '#' %></li>
              </ul>
            </nav>
          </div>
        </header>
        <div class="container">
          <%= yield %>
        </div>
      </body>
    </html>

> app/views/static_pages/home.html.erb

    <div class="center jumbotron">
      <h1>Welcome to the Sample App</h1>
    
      <h2>
        This is the home page.
      </h2>
    
      <%= link_to "Sign up now!", '#', class: "btn btn-lg btn-primary" %>
    </div>
    
    <%= link_to image_tag("rails.png", alt: "Rails logo"),
                'http://rubyonrails.org/' %>



