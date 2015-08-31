2 - peep_app
-----------

**2.1 Planejando a aplicação**

    $ cd ~/curso-rails
    $ bundle exec rails _4.2.3_ new peep_app

**2.2 User model**

- id (integer) identificador único do usuário
- name (string) nome público do usuário
- email (string) que também servirá para login.

**2.3 Micropost model**

- id (integer)
- content (text)
- user_id (integer)

**2.4 Resource Users**

Um resource é a implementação de um data model e de uma interface web para esse model. Essa combinação nos permite pensar em users como um objeto que pode ser criado, lido, editado e excluído através do protocolo web HTTP.

    $ ./bin/rails generate scaffold User name:string email:string

para a aplicação funcionar temos que rodar a migração de banco de dados usando o comando rake:

    $ ./bin/rake db:migrate

**2.5 User Tour**

- /users -> index -> lista todos os users
- /users/1 -> show -> mostra o user que tem id igual a 1
- /users/new -> new -> página para criar um novo user
- /users/1/edit -> edit -> edita um user que tem id igual a 1

**2.6 MVC em ação**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/mvc_detailed.png)

- O browser envia uma requisição para a URL /users.
- O Rails direciona a rota /users para a action index do controller Users.
- A action index solicita ao User model que busque todos os users (User.all).
- O User model recupera todos os users do banco de dados.
- O User model retorna uma lista de users ao controller.
- O controller captura os users na variável @users, que é passada para a view index.
-  A view usa ruby  inserido na página para renderizar o HTML.
- O controller devolve o HTML para o browser.

vamos apontar a raiz da aplicação para a listagem de users:

    Rails.application.routes.draw do
      resources :users
      root 'users#index'
    end

**O Que é REST?**

A Representational State Transfer (REST), em português Transferência de Estado Representacional, é uma abstração da arquitetura da World Wide Web, mais precisamente, é um estilo arquitetural que consiste de um conjunto coordenado de restrições arquiteturais aplicadas a componentes, conectores e elementos de dados dentro de um sistema de hipermídia distribuído. O REST ignora os detalhes da implementação de componente e a sintaxe de protocolo com o objetivo de focar nos papéis dos componentes, nas restrições sobre sua interação com outros componentes e na sua interpretação de elementos de dados significantes.

**2.7 Pontos fracos desse Users resource**

- Falta validação;
- Não existe autenticação;
- Não foram implementados testes;
- Falta de estilos e layout;

**2.8 O resource microposts**

    $ ./bin/rails generate scaffold Micropost content:text user_id:integer
    $ ./bin/rake db:migrate

**2.9 Fazendo o micropost ser micro**

Vamos implementar uma constraint para aceitar microposts com no máximo 140 caracteres (à la Twitter)

> app/models/micropost.rb

    class Micropost < ActiveRecord::Base
      validates :content, length: { maximum: 140 }
    end


**2.10 user has_many microposts**

> app/models/user.rb

    class User < ActiveRecord::Base
      has_many :microposts
    end

> app/models/micropost.rb

    class Micropost < ActiveRecord::Base
      belongs_to :user
      validates :content, length: { maximum: 140 }
    end

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/micropost_user_association.png)

Vamos verificar no console se essa associação funciona:

      $ ./bin/rails console      
      >> first_user = User.first      
      >> first_user.microposts
      >> micropost = first_user.microposts.first #Micropost.first também funcionaria nesse caso.
      >> micropost.user
      >> exit

**2.11 Herança e hierarquia**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_model_inheritance.png)



![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_controller_inheritance.png)



**2.12 Exercício**

Alterar a validação no micropost para para garantir que ele não seja criado em branco:

    class Micropost < ActiveRecord::Base
        belongs_to :user
        validates :content, length: { maximum: 140 },
                            presence: true
    end

Adicionar a mesma validação nos campos nome e email do User model.


