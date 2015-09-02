
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
      This is the home page.
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

o comando puts automaticamente um \n na saída. Ja o metodo print, não:

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

por esse motivo strings de aspas simples são muito úteis quando se quer utilizar um valor literal, por exemplo:

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


Como nil é um objeto ele responde ao método to_s que é capaz de converter praticamente qualquer objeto em uma string:

    >> nil.to_s
    => ""
    >> nil.empty?
    NoMethodError: undefined method `empty?' for nil:NilClass
    >> nil.to_s.empty?      # Message chaining
    => true

Existe um metodo especial para testar se um objeto é nil:

    >> "foo".nil?
    => false
    >> "".nil?
    => false
    >> nil.nil?
    => true

A linha de código:

    puts "x is not empty" if !x.empty?

mostra que, em ruby, é possivel usar o if para executar algo somente se a condição posterior ao if for atendida. unless funciona de forma similar:

    >> string = "foobar"
    >> puts "The string '#{string}' is nonempty." unless string.empty?
    The string 'foobar' is nonempty.
    => nil

O objeto nil é especial uma vez que ele e o unico objeto de ruby, em um contexto booleano, considerado falso(além, claro do proprio false):

    >> !!nil
    => false

Todos os outros objetos, incluindo 0, são verdadeiros:

    >> !!0
    => true

**Definição de métodos**

    >> def string_message(str = '')
    >>   if str.empty?
    >>     "It's an empty string!"
    >>   else
    >>     "The string is nonempty."
    >>   end
    >> end
    => :string_message
    >> puts string_message("foobar")
    The string is nonempty.
    >> puts string_message("")
    It's an empty string!
    >> puts string_message
    It's an empty string!

**De volta ao title helper**

    module ApplicationHelper
    
      # Returns the full title on a per-page basis.       # Documentation comment
      def full_title(page_title = '')                     # Method def, optional arg
        base_title = "Ruby on Rails Tutorial Sample App"  # Variable assignment
        if page_title.empty?                              # Boolean test
          base_title                                      # Implicit return
        else
          page_title + " | " + base_title                 # String concatenation
        end
      end
    end

**arrays e ranges**

    >>  "foo bar     baz".split     # Split a string into a three-element array.
    => ["foo", "bar", "baz"]
    >> "fooxbarxbazx".split('x')
    => ["foo", "bar", "baz"]
    >> a = [42, 8, 17]
    => [42, 8, 17]
    >> a[0]               # Ruby uses square brackets for array access.
    => 42
    >> a[1]
    => 8
    >> a[2]
    => 17
    >> a[-1]              # Indices can even be negative!
    => 17
    >> a.first
    => 42
    >> a.second
    => 8
    >> a.last
    => 17
    >> a.last == a[-1]    # Comparison using ==
    => true

arrays respondem a vários métodos:

    >> a
    => [42, 8, 17]
    >> x = a.length
    => 3
    >> a.empty?
    => false
    >> a.include?(42)
    => true
    >> a.sort
    => [8, 17, 42]
    >> a.reverse
    => [17, 8, 42]
    >> a.shuffle
    => [17, 42, 8]
    >> a
    => [42, 8, 17]

Repare que nenhum dos métodos acima altera o objeto a propriamente dito. Para mudar o array precisamos usar o sinal de exclamação no final do método:

    >> a
    => [42, 8, 17]
    >> a.sort!
    => [8, 17, 42]
    >> a
    => [8, 17, 42]

Também é possível adicionar elementos em um array usando o método push ou seu operador equivalente, <<:

    >> a.push(6)                  # Pushing 6 onto an array
    => [42, 8, 17, 6]
    >> a << 7                     # Pushing 7 onto an array
    => [42, 8, 17, 6, 7]
    >> a << "foo" << "bar"        # Chaining array pushes
    => [42, 8, 17, 6, 7, "foo", "bar"]

o metodo join faz o servico contrário do split:

    >> a
    => [42, 8, 17, 7, "foo", "bar"]
    >> a.join                       # Join on nothing.
    => "428177foobar"
    >> a.join(', ')                 # Join on comma-space.
    => "42, 8, 17, 7, foo, bar"

Ranges são muito utilizados em conjunto com arrays:

    >> 0..9
    => 0..9
    >> 0..9.to_a              # Oops, call to_a on 9.
    NoMethodError: undefined method `to_a' for 9:Fixnum
    >> (0..9).to_a            # Use parentheses to call to_a on the range.
    => [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

Ranges são muito úteis para extrair partes de arrays:

    >> a = %w[foo bar baz quux]         # Use %w to make a string array.
    => ["foo", "bar", "baz", "quux"]
    >> a[0..2]
    => ["foo", "bar", "baz"]
    >> a = (0..9).to_a
    => [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    >> a[2..(a.length-1)]               # Explicitly use the array's length.
    => [2, 3, 4, 5, 6, 7, 8, 9]
    >> a[2..-1]                         # Use the index -1 trick.
    => [2, 3, 4, 5, 6, 7, 8, 9]

Ranges tambem funcionam com letras:

    >> ('a'..'e').to_a
    => ["a", "b", "c", "d", "e"]

**Blocks**

Arrays and ranges respondem a vários métodos que aceitam blocos:

    >> (1..5).each { |i| puts 2 * i }
    2
    4
    6
    8
    10
    => 1..5

As chaves são indicadores de bloco, mas existe uma sintaxe alternativa:

    >> (1..5).each do |i|
    ?>   puts 2 * i
    >> end
    2
    4
    6
    8
    10
    => 1..5

blocos podem ter mais de uma linha:

    >> (1..5).each do |number|
    ?>   puts 2 * number
    >>   puts '--'
    >> end
    2
    --
    4
    --
    6
    --
    8
    --
    10
    --
    => 1..5

vejamos outros exemplos:

    >> 3.times { puts "Betelgeuse!" }   # 3.times takes a block with no variables.
    "Betelgeuse!"
    "Betelgeuse!"
    "Betelgeuse!"
    => 3
    >> (1..5).map { |i| i**2 }          # The ** notation is for 'power'.
    => [1, 4, 9, 16, 25]
    >> %w[a b c]                        # Recall that %w makes string arrays.
    => ["a", "b", "c"]
    >> %w[a b c].map { |char| char.upcase }
    => ["A", "B", "C"]
    >> %w[A B C].map { |char| char.downcase }
    => ["a", "b", "c"]

As duas últimas expressões acima pode também ser escritas assim:

    >> %w[A B C].map { |char| char.downcase }
    => ["a", "b", "c"]
    >> %w[A B C].map(&:downcase)
    => ["a", "b", "c"]

**Hashes e symbols**

Hashes sao, basicamente, arrays que podem ter indices que não são inteiros. Na verdade os índices , ou keys, podem ser praticamente qualquer tipo de objeto, por exemplo strings:

    >> user = {}                          # {} is an empty hash.
    => {}
    >> user["first_name"] = "Lucas"     # Key "first_name", value "Lucas"
    => "Lucas"
    >> user["last_name"] = "Pratto"        # Key "last_name", value "Pratto"
    => "Pratto"
    >> user["first_name"]                 # Element access is like arrays.
    => "Lucas"
    >> user                               # A literal representation of the hash
    => {"last_name"=>"Lucas", "first_name"=>"Pratto"}

Podemos definir mais de um elemento de uma vez usando keys e valores separados por =>:

    >> user = { "first_name" => "Lucas", "last_name" => "Pratto" }
    => {"last_name"=>"Lucas", "first_name"=>"Pratto"}

Até agora usamos strings como chaves de hash, mas o mais comum em ruby é utilizar simbolos. Símbolos se parecem com strings, mas são precedidos por dois pontos e não são cercados por aspas:

    >> "name".split('')
    => ["n", "a", "m", "e"]
    >> :name.split('')
    NoMethodError: undefined method `split' for :name:Symbol
    >> "foobar".reverse
    => "raboof"
    >> :foobar.reverse
    NoMethodError: undefined method `reverse' for :foobar:Symbol

direfente de strings , nem todos os caracteres são permitidos:

    >> :foo-bar
    NameError: undefined local variable or method `bar' for main:Object
    >> :2foo
    SyntaxError

usando símbolos podemos definir um hash assim:

    >> user = { :name => "Lucas Pratto", :email => "pratto@gnv.com" }
    => {:name=>"Lucas Pratto", :email=>"pratto@gnv.com"}
    >> user[:name]              # Access the value corresponding to :name.
    => "Lucas Pratto"
    >> user[:password]          # Access the value of an undefined key.
    => nil

existe uma sintaxe especial para usar símbolos em hashes:

    >> h1 = { :name => "Lucas Pratto", :email => "pratto@gnv.com" }
    => {:name=>"Lucas Pratto", :email=>"pratto@gnv.com"}
    >> h2 = { name: "Lucas Pratto", email: "pratto@gnv.com" }
    => {:name=>"Lucas Pratto", :email=>"pratto@gnv.com"}
    >> h1 == h2
    => true

Hahes podem conter outros hashes:

    >> params = {}        # Define a hash called 'params' (short for 'parameters').
    => {}
    >> params[:user] = { name: "Lucas Pratto", email: "pratto@gnv.com" }
    => {:name=>"Lucas Pratto", :email=>"pratto@gnv.com"}
    >> params
    => {:user=>{:name=>"Lucas Pratto", :email=>"pratto@gnv.com"}}
    >>  params[:user][:email]
    => "pratto@gnv.com"

**Ruby classes**

O Ruby, como muitas linguagens orientadas a objetos, usa classes para organizar métodos; essas classe podem então ser instanciadas para criar objetos.

Vamos criar uma classe chamada Word com um método chamado palindrome?:

    >> class Word
    >>   def palindrome?(string)
    >>     string == string.reverse
    >>   end
    >> end
    => :palindrome?

Podemos usar nossa classe assim:

    >> w = Word.new              # Make a new Word object.
    => #<Word:0x22d0b20>
    >> w.palindrome?("foobar")
    => false
    >> w.palindrome?("level")
    => true

Já que word é uma string, faz mais sentido que nossa classe herde de String:

    >> class Word < String             # Word inherits from String.
    >>   # Returns true if the string is its own reverse.
    >>   def palindrome?
    >>     self == self.reverse        # self is the string itself.
    >>   end
    >> end
    => nil
    >> s = Word.new("level")    # Make a new Word, initialized with "level".
    => "level"
    >> s.palindrome?            # Words have the palindrome? method.
    => true
    >> s.length                 # Words also inherit all the normal string methods.
    => 5

**Modificando classes existentes**

    >> "level".palindrome?
    NoMethodError: undefined method `palindrome?' for "level":String

    >> class String
    >>   # Returns true if the string is its own reverse.
    >>   def palindrome?
    >>     self == self.reverse
    >>   end
    >> end
    => nil
    >> "mussum".palindrome?
    => true























