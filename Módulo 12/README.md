

12 - Internacionalização
-------------------

[Traduções para vários idiomas podem ser encontrados aqui](https://github.com/svenfuchs/rails-i18n)

**A API I18n**

Os métodos mais importantes da api I18n são:

    translate # Lookup text translations
    localize  # Localize Date and Time objects to local formats

ambos possuem aliases #t e #l:

    I18n.t 'store.title'
    I18n.l Time.now

**Preparando a aplicação para internacionalização**

O Rails carrega todos os arquivos .rb e .yml da pasta config/locales directory automaticamente. O aquivo en.yml que foi criado pelo gerador da aplicação contém o seguinte:

    en:
      hello: "Hello world"

O idioma padrão e o caminho de carregamento dos arquivos de tradução podem ser configurados no arquivo application.rb:

    # The default locale is :en and all translations from config/locales/*.rb,yml are auto loaded.
    # config.i18n.load_path += Dir[Rails.root.join('my', 'locales', '*.{rb,yml}').to_s]
    # config.i18n.default_locale = :de

**suporte a vários idiomas**

> app/controllers/application_controller.rb

    before_action :set_locale
    .
    .
    .     
    def set_locale
      I18n.locale = params[:locale] || I18n.default_locale
    end

com isso podemos ver a aplicação traduzida passando um parâmetro pela url, dessa maneira: http://localhost:3000?locale=pt-BR

**Setando o Locale a partir do Domínio ou subdomínio**

> app/controllers/application_controller.rb

    before_action :set_locale
     
    def set_locale
      I18n.locale = extract_locale_from_tld || I18n.default_locale
    end
     
    # Get locale from top-level domain or return nil if such locale is not available
    # You have to put something like:
    #   127.0.0.1 application.com
    #   127.0.0.1 application.com.br
    # in your /etc/hosts file to try this out locally
    def extract_locale_from_tld
      parsed_locale = request.host.split('.').last
      parsed_locale = "pt-BR" if parsed_locale == "br"
      I18n.available_locales.map(&:to_s).include?(parsed_locale) ? parsed_locale : nil
    end

também podemos pegar o locale pelo subdomínio de maneira similar:

    # Get locale code from request subdomain (like http://it.application.local:3000)
    # You have to put something like:
    #   127.0.0.1 gr.application.local
    # in your /etc/hosts file to try this out locally
    def extract_locale_from_subdomain
      parsed_locale = request.subdomains.first
      parsed_locale = "pt-BR" if parsed_locale == "br"
      I18n.available_locales.map(&:to_s).include?(parsed_locale) ? parsed_locale : nil
    end

[Rails Internationalization](http://guides.rubyonrails.org/i18n.html)



