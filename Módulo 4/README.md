
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

**Printing**

    >> puts "foo"     # put string
    foo
    => nil

o comando puts automaticamente um \n na saida. Ja o metodo print, nao:

    >> print "foo"    # print string (same as puts, but without the newline)
    foo=> nil
    >> print "foo\n"  # Same as puts "foo"
    foo
    => nil

**Strings com aspas simples**

Aparentemente aspas duplas e simples são a mesma coisa:

    >> 'foo'          # A single-quoted string
    => "foo"
    >> 'foo' + 'bar'
    => "foobar"

Mas existe uma diferença importante; Não é possível usar interpolação em strings de aspas simples:

    >> '#{foo} bar'     # Single-quoted strings don't allow interpolation
    => "\#{foo} bar"

por esse motivo strings de aspas simples sao muito uteis quando se quer utilizar um valor literal, por exemplo:

    >> '\n'       # A literal 'backslash n' combination
    => "\\n"

**Objetos e passagem de mensagens**

Tudo em Ruby, incluindo strings e até mesmo nil, é um objeto.
Um object do tipo string, por exemplo, responde à mensagem length, que retorna o número de caracteres da string:

    >> "foobar".length        # Passing the "length" message to a string
    => 6

Normalmente as mensagens que são passadas para os objetos são métodos, que são funções definidas nesses objetos. Um objeto string também responde ao método empty?:

    >> "foobar".empty?
    => false
    >> "".empty?
    => true

O sinal de interrogação no final do método empty? é uma convenção do ruby para indicar que o valor de retorno é um booleano:

    >> s = "foobar"
    >> if s.empty?
    >>   "The string is empty"
    >> else
    >>   "The string is nonempty"
    >> end
    => "The string is nonempty"

Podemos incluir mais de um teste usando elsif (else + if):

    >> if s.nil?
    >>   "The variable is nil"
    >> elsif s.empty?
    >>   "The string is empty"
    >> elsif s.include?("foo")
    >>   "The string includes 'foo'"
    >> end
    => "The string includes 'foo'"

Booleanos podem ser combinados usando os operadores && (“and”), || (“or”) e ! (“not”):

    >> x = "foo"
    => "foo"
    >> y = ""
    => ""
    >> puts "Both strings are empty" if x.empty? && y.empty?
    => nil
    >> puts "One of the strings is empty" if x.empty? || y.empty?
    "One of the strings is empty"
    => nil
    >> puts "x is not empty" if !x.empty?
    "x is not empty"
    => nil


Como nil e um objeto ele responde ao metodo to_s que e capaz de converter praticamente qualquer objeto em uma string:

    >> nil.to_s
    => ""
    >> nil.empty?
    NoMethodError: undefined method `empty?' for nil:NilClass
    >> nil.to_s.empty?      # Message chaining
    => true

Existe um metodo especial para testar se um objeto e nil:

    >> "foo".nil?
    => false
    >> "".nil?
    => false
    >> nil.nil?
    => true

A linha de codigo:

    puts "x is not empty" if !x.empty?

mostra que, em ruby, e possivel usar o if para executar algo somente se a condiçao posterior ao if for atendida. unless funciona de forma similar:

    >> string = "foobar"
    >> puts "The string '#{string}' is nonempty." unless string.empty?
    The string 'foobar' is nonempty.
    => nil

O objeto nil e especial uma vez que ele e o unico objeto de ruby, em um contexto booleano, considerado falso(alem, claro do proprio false):

    >> !!nil
    => false

Todos os outros objetos, incluindo 0, sao verdadeiros:

    >> !!0
    => true

**Definindo métodos**













