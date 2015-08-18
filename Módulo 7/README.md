

7 - Cadastro de Usuário
-----------------------

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/profile_mockup_profile_name_bootstrap.png)

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/profile_mockup_bootstrap.png)

**Debug e Rails environments**

> app/views/layouts/application.html.erb

    <!DOCTYPE html>
    <html>
      .
      .
      .
      <body>
        <%= render 'layouts/header' %>
        <div class="container">
          <%= yield %>
          <%= render 'layouts/footer' %>
          <%= debug(params) if Rails.env.development? %>
        </div>
      </body>
    </html>

O Rails tem três ambientes configurados disponíveis: test, development, e production. O ambiente padrão do Rails console é o development:

      $ ./bin/rails console
      Loading development environment
      >> Rails.env
      => "development"
      >> Rails.env.development?
      => true
      >> Rails.env.test?
      => false

É possível rodar o console com em ambiente diferente, basta passar um parâmetro:

      $ ./bi/rails console test
      Loading test environment
      >> Rails.env
      => "test"
      >> Rails.env.test?
      => true

Assim como no console, development é o ambiente padrão do Rails server, mas também pode ser iniciado com outro ambiente:

    $ ./bin/rails server --environment production

Para que a aplicação funcione em modo produção é preciso criar o banco de dados:

      $ ./bin/rake db:migrate RAILS_ENV=production


