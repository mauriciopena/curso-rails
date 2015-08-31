
**11 - Seguindo usuários**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_profile_mockup_3rd_edition.png)

o usuário calvin vai até a tela de listagem de usuários onde pode encontrar outro usuário para seguir:

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_user_index_mockup_bootstrap.png)

ele escolhe o usuário hobbes:

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_other_profile_follow_button_mockup_3rd_edition.png)

a contagem de followers do usuário hobbes aumenta e o botão "follow" vira "unfollow"

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_other_profile_unfollow_button_mockup_3rd_edition.png)

voltando para a homepage o usuário calvin agora também encontra listados os microposts do usuário hobbes

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/page_flow_home_page_feed_mockup_3rd_edition.png)


**Proposta de modelo de relacionamento**

![enter image description here](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/naive_user_has_many_following.png)

problemas dessa proposta:

 - redundância de dados 
 - duas tabelas, com redundância de dados: followers e following
 - manter os dados de usuário atualizados seria muito difícil



