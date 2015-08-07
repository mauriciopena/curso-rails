2 - peep app
-----------

**2.1 Planejando a aplicação**

    $ cd ~/curso-rails
    $ mkdir peep
    $ cd peep
    $ rails _4.2.3_ new peep_app

**2.2 User model**

"Users" terão os seguintes atributos: 

- id (integer) identificador único do usuário
- name (string) nome público do usuário
- email (string) que também servirá para login.

**2.3 Micropost model**

- id (integer)
- content (text)
- user_id (integer)

**2.4 Resource Users**

Um resource é a implementação de um data model e de uma interface web para esse model. Essa combinação nos permite pensar em users como um objeto que pode ser criado, lido, editado e excluído através do protocolo web HTTP.

    $ rails generate scaffold User name:string email:string



