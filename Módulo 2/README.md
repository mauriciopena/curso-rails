2 - peep app
-----------

**2.1 Planejando a aplicação**

    $ cd ~/curso-rails
    $ mkdir peep
    $ cd peep
    $ rails _4.2.3_ new peep_app

**2.2 User model**
Os "Users" da nossa aplicação terão os seguintes atributos: 
- id (integer) identificador único do usuário
- name (string) nome público do usuário
- email (string) que também servirá para login.

**2.3 Micropost model**

- id (integer)
- content (text)
- user_id (integer)
