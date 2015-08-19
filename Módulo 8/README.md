

8 - Login e Logout
------------------

    $ ./bin/rails generate controller Sessions new

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
      resources :users
    end

**Login Form**


> app/views/sessions/new.html.erb

    <% provide(:title, "Log in") %>
    <h1>Log in</h1>
    
    <div class="row">
      <div class="col-md-6 col-md-offset-3">
        <%= form_for(:session, url: login_path) do |f| %>
    
          <%= f.label :email %>
          <%= f.email_field :email, class: 'form-control' %>
    
          <%= f.label :password %>
          <%= f.password_field :password, class: 'form-control' %>
    
          <%= f.submit "Log in", class: "btn btn-primary" %>
        <% end %>
    
        <p>New user? <%= link_to "Sign up now!", signup_path %></p>
      </div>
    </div>

**Buscando e autenticando um usuário**





