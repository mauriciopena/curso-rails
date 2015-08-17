

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

**Bootstrap**

http://getbootstrap.com/

> Gemfile

    gem 'bootstrap-sass'

instalar a gem:

    $ bundle install

vamos criar um arquivo para conter nosso código css:

    $ touch app/assets/stylesheets/custom.css.scss

> app/assets/stylesheets/custom.css.scss

    @import "bootstrap-sprockets";
    @import "bootstrap";

> app/assets/stylesheets/custom.css.scss

    @import "bootstrap-sprockets";
    @import "bootstrap";
    
    /* universal */
    
    body {
      padding-top: 60px;
    }
    
    section {
      overflow: auto;
    }
    
    textarea {
      resize: vertical;
    }
    
    .center {
      text-align: center;
    }
    
    .center h1 {
      margin-bottom: 10px;
    }

    /* typography */
    
    h1, h2, h3, h4, h5, h6 {
      line-height: 1;
    }
    
    h1 {
      font-size: 3em;
      letter-spacing: -2px;
      margin-bottom: 30px;
      text-align: center;
    }
    
    h2 {
      font-size: 1.2em;
      letter-spacing: -1px;
      margin-bottom: 30px;
      text-align: center;
      font-weight: normal;
      color: #777;
    }
    
    p {
      font-size: 1.1em;
      line-height: 1.7em;
    }
    
    /* header */
    
    #logo {
      float: left;
      margin-right: 10px;
      font-size: 1.7em;
      color: #fff;
      text-transform: uppercase;
      letter-spacing: -1px;
      padding-top: 9px;
      font-weight: bold;
    }
    
    #logo:hover {
      color: #fff;
      text-decoration: none;
    }

**Partials**

> app/views/layouts/application.html.erb

    <!DOCTYPE html>
    <html>
      <head>
        <title><%= full_title(yield(:title)) %></title>
        <%= stylesheet_link_tag 'application', media: 'all',
                                               'data-turbolinks-track' => true %>
        <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
        <%= csrf_meta_tags %>
        <%= render 'layouts/shim' %>
      </head>
      <body>
        <%= render 'layouts/header' %>
        <div class="container">
          <%= yield %>
        </div>
      </body>
    </html>

> app/views/layouts/_shim.html.erb

    <!--[if lt IE 9]>
      <script src="//cdnjs.cloudflare.com/ajax/libs/html5shiv/r29/html5.min.js">
      </script>
    <![endif]-->

> app/views/layouts/_header.html.erb

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

> app/views/layouts/_footer.html.erb

    <footer class="footer">
      <small>
        Curso de Ruby on Rails
      </small>
      <nav>
        <ul>
          <li><%= link_to "About",   '#' %></li>
          <li><%= link_to "Contact", '#' %></li>      
        </ul>
      </nav>
    </footer>

> app/views/layouts/application.html.erb

    <!DOCTYPE html>
    <html>
      <head>
        <title><%= full_title(yield(:title)) %></title>
        <%= stylesheet_link_tag "application", media: "all",
                                               "data-turbolinks-track" => true %>
        <%= javascript_include_tag "application", "data-turbolinks-track" => true %>
        <%= csrf_meta_tags %>
        <%= render 'layouts/shim' %>
      </head>
      <body>
        <%= render 'layouts/header' %>
        <div class="container">
          <%= yield %>
          <%= render 'layouts/footer' %>
        </div>
      </body>
    </html>

> app/assets/stylesheets/custom.css.scss

    .
    .
    .
    /* footer */
    
    footer {
      margin-top: 45px;
      padding-top: 5px;
      border-top: 1px solid #eaeaea;
      color: #777;
    }
    
    footer a {
      color: #555;
    }
    
    footer a:hover {
      color: #222;
    }
    
    footer small {
      float: left;
    }
    
    footer ul {
      float: right;
      list-style: none;
    }
    
    footer ul li {
      float: left;
      margin-left: 15px;
    }

**Sass e a asset pipeline**

Nas versões do Rails 3.0 e anteriores, os assets estáticos ficavam no diretório public/ 

    public/stylesheets
    public/javascripts
    public/images 

Na versão atual do Rails, existem três diretórios para esse fim:

    app/assets: assets específicos para a aplicação
    lib/assets: assets para pacotes desenvolvidos pela equipe de desenvolvimento
    vendor/assets: assets desenvolvidos por terceiros

**arquivo Manifest**

Esses arquivos são usados para que o Rails, através da gem sprokets, faça a combinação dos assets para formar arquivos únicos.

> app/assets/stylesheets/application.css

    /*
     * This is a manifest file that'll be compiled into application.css, which
     * will include all the files listed below.
     *
     * Any CSS and SCSS file within this directory, lib/assets/stylesheets,
     * vendor/assets/stylesheets, or vendor/assets/stylesheets of plugins, if any,
     * can be referenced here using a relative path.
     *
     * You're free to add application-wide styles to this file and they'll appear
     * at the bottom of the compiled file so the styles you add here take
     * precedence over styles defined in any styles defined in the other CSS/SCSS
     * files in this directory. It is generally better to create a new file per
     * style scope.
     *
     *= require_tree .
     *= require_self
     */

**Engines de pré-processamento**

O Rails sabe qual processador passar um arquivo pela extensão do mesmo. Os mais comuns são .scss para Sass, .coffee para CoffeeScript, e .erb para Ruby (ERb).

Pré-processadores podem ser encadeados, por exemplo:

    foobar.js.coffee

passa pelo processador de coffeescript, e 

    foobar.js.erb.coffee

passa pelo CoffeeScript primeiro e depois ERb.




