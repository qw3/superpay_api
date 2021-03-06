# SuperPay API - Gateway de pagamento

## Versão Beta

Gem para utilização do Gateway de pagamento [SuperPay API](http://superpay.locaweb.com.br/).

A biblioteca SuperPayAPI em Ruby é um conjunto de classes de domínio que facilitam, para o desenvolvedor Ruby, a utilização das funcionalidades que o SuperPay oferece na forma de [APIs](http://wiki.superpay.com.br/wikiSuperPay/index.php/P%C3%A1gina_principal). Com a biblioteca instalada e configurada, você pode facilmente integrar funcionalidades como:

 - [Criar transação](http://wiki.superpay.com.br/wikiSuperPay/index.php/Criar_transa%C3%A7%C3%A3o_SOAP)
 - [Consultar transação](http://wiki.superpay.com.br/wikiSuperPay/index.php/Consultar_transa%C3%A7%C3%A3o_SOAP)
 - [Capturar transação](http://wiki.superpay.com.br/wikiSuperPay/index.php/Capturar_transa%C3%A7%C3%A3o_SOAP)
 - [Cancelar transação](http://wiki.superpay.com.br/wikiSuperPay/index.php/Cancelar_transa%C3%A7%C3%A3o_SOAP)
 - [Estorno de transação](http://wiki.superpay.com.br/wikiSuperPay/index.php/Estorno_de_transa%C3%A7%C3%A3o_SOAP)
 - [Campainha](http://wiki.superpay.com.br/wikiSuperPay/index.php/Campainha)
 - [Múltiplos Cartões](http://wiki.superpay.com.br/wikiSuperPay/index.php/M%C3%BAltiplos_Cart%C3%B5es) \(Em desenvolvimento\)
 - [One Click](http://wiki.superpay.com.br/wikiSuperPay/index.php/One_Click) \(Em desenvolvimento\)
 - [Cobrança Recorrente](http://wiki.superpay.com.br/wikiSuperPay/index.php/Cobran%C3%A7a_Recorrente) \(Em desenvolvimento\)

## Instalando

### Gemfile

Adicione esta linha ao Gemfile do seu aplicativo:

```ruby
  gem 'superpay_api'
```

 - Execute o comando `bundle install`.

### Instalação direta

```ruby
  gem install superpay_api
```

## Configuração

Para fazer a autenticação, você precisará configurar as credenciais do SuperPay. Crie o arquivo `config/initializers/superpay_api.rb` com o conteúdo abaixo.

```ruby
SuperpayApi.configure do |config|

  #
  # Código que identifica o estabelecimento dentro do SuperPay
  # Enviado pelo SuperPay
  config.estabelecimento = "1010101010101010"

  #
  # Código que identifica o usuario dentro do SuperPay
  # Enviado pelo SuperPay
  config.usuario = "superpay"

  #
  # Código que identifica a senha dentro do SuperPay
  # Enviado pelo SuperPay
  config.senha = "superpay"

  #
  # Opções de ambiente
  # [:homologacao, :producao]
  config.ambiente = :producao

  #
  # URL será sempre acionada quando o status do pedido mudar.
  # Deve estar preparada para receber dados de campainha
  config.url_campainha = 'http://localhost:3000'

  #
  # Opções de idiomas
  # [:portugues, :ingles, :espanhol]
  config.idioma = :portugues

  #
  # Opções de Origem da Transação
  # Consulte área de suporte sobre a habilitação das origens
  # [:ecommerce, :mobile, :ura, :pos]
  config.origem_transacao = :ecommerce
end
```

## Criar transação

A principal funcionalidade do SuperPay. Envia a transação e realiza o pagamento com base na mesma. Uma das características mais importantes desse processo é uma conferência realizada pelo SuperPay na operadora antes de enviar a transação.

Antes do envio da transação para as operadoras e instituições financeiras, o gateway de pagamento realiza uma pequena conferência na mesma para se certificar que não existe nenhuma transação com o mesmo número de pedido para aquele estabelecimento.

O número do pedido é um valor enviado pelo próprio estabelecimento, portanto é importante que o mesmo se certifique que tais valores sejam únicos para cada transação. Caso essa consulta prévia do gateway identifique na operadora uma transação já enviada a nova transação com o mesmo número de pedido pode resultar em falha.

Para iniciar uma requisição de pagamento, você precisa instanciar a classe `SuperpayApi::Transacao`. Isso normalmente será feito em seu controller de checkout.

A classe `SuperpayApi::Transacao` é montada com outras classes \(`SuperpayApi::DadosUsuario`, `SuperpayApi::Endereco`, `SuperpayApi::Telefone`, `SuperpayApi::ItemPedido`\) com suas devidas validações.

O retorno será um objeto da classe `SuperpayApi::Retorno` \(Verifique o objeto mais abaixo\). Tal objeto possui todas as informações necessárias para validar o resultado da transação.

Importante realizar a validação do objeto antes de enviar o pagamento, para isso verifique a [documentação](http://www.rubydoc.info/github/qw3/superpay_api/).

### Importante ressaltar que todos os campos deste objeto são obrigatórios em caso de utilização de análise de fraude/risco.

### Montar [Transação](http://www.rubydoc.info/github/qw3/superpay_api/SuperpayApi/Transacao)

```ruby
transacao = SuperpayApi::Transacao.new ({
  numero_transacao:               "1",
  ip:                             "127.0.0.1",
  codigo_forma_pagamento:         120,
  valor:                          "28260",
  valor_desconto:                 "0",
  taxa_embarque:                  "0",
  parcelas:                       "1",
  nome_titular_cartao_credito:    "Visa",
  numero_cartao_credito:          "4444333322221111",
  codigo_seguranca:               "123",
  data_validade_cartao:           "12/2017",
  # vencimento_boleto:              self.vencimento_boleto,
  # url_redirecionamento_pago:      self.url_redirecionamento_pago,
  # url_redirecionamento_nao_pago:  self.url_redirecionamento_nao_pago,
  # campo_livre1:                   self.campo_livre1,
  # campo_livre2:                   self.campo_livre2,
  # campo_livre3:                   self.campo_livre3,
  # campo_livre4:                   self.campo_livre4,
  # campo_livre5:                   self.campo_livre5,
  # dados_usuario_transacao:        dados_usuario, # Objeto da classe SuperpayApi::DadosUsuario
  # itens_do_pedido:                [item_pedido_1, item_pedido_2], # Array de objetos da classe SuperpayApi::ItemPedido
})

# Após montar o objeto e verificar as validações, chame a função enviar_pagamento
retorno = transacao.enviar_pagamento if transacao.valid?
```

### Montar [Itens do Pedido](http://www.rubydoc.info/github/qw3/superpay_api/SuperpayApi/ItemPedido)

```ruby
item_pedido_1 = SuperpayApi::ItemPedido.new ({
  codigo_produto:           "11",
  nome_produto:             "Item Pedido 1",
  quantidade_produto:       "2",
  valor_unitario_produto:   "12312",
  codigo_categoria:         "1",
  nome_categoria:           "Itens de Pedidos",
})

item_pedido_2 = SuperpayApi::ItemPedido.new ({
  codigo_produto:           "12",
  nome_produto:             "Item Pedido 2",
  quantidade_produto:       "3",
  valor_unitario_produto:   "1212",
  codigo_categoria:         "1",
  nome_categoria:           "Itens de Pedidos",
})
```

### Montar [Dados do Usuário](http://www.rubydoc.info/github/qw3/superpay_api/SuperpayApi/DadosUsuario)

```ruby
dados_usuario = SuperpayApi::DadosUsuario.new ({
  codigo_cliente:                 1,
  tipo_cliente:                   :pessoa_fisica,
  nome:                           "Leandro Falcão",
  documento:                      "999.999.999-9",
  documento_2:                    "99.999.999-9",
  sexo:                           :masculino,
  data_nascimento:                "01/01/1980",
  email:                          "contato@qw3.com.br",
  endereco_comprador:              endereco, # Objeto da classe SuperpayApi::Endereco
  endereco_entrega:                endereco, # Objeto da classe SuperpayApi::Endereco
  telefone_comprador:              telefone, # Objeto da classe SuperpayApi::Telefone
  # telefone_adicional_comprador:  telefone, # Objeto da classe SuperpayApi::Telefone
  telefone_entrega:                telefone, # Objeto da classe SuperpayApi::Telefone
  # telefone_adicional_entrega:    telefone, # Objeto da classe SuperpayApi::Telefone
})
```

### Montar [Endereço](http://www.rubydoc.info/github/qw3/superpay_api/SuperpayApi/Endereco)

```ruby
endereco = SuperpayApi::Endereco.new ({
  logradouro:    "Rua Dom Pedro II",
  numero:        "1330",
  bairro:        "Vila Monteiro (Gleba I)",
  complemento:   "Sala A",
  cidade:        "São Carlos",
  estado:        "SP",
  cep:           "13560-320",
})
```

### Montar [Telefone](http://www.rubydoc.info/github/qw3/superpay_api/SuperpayApi/Telefone)

```ruby
telefone = SuperpayApi::Telefone.new ({
  codigo_tipo_telefone:   :comercial,
  telefone:               "3416-6404",
  ddd:                    "16",
  ddi:                    "+55",
})
```

## Consultar transação

O SuperPay disponibiliza um método para consulta de pedidos. Através dele é possível verificar a situação atual de uma transação, verificando, por exemplo, o status em que o pedido se encontra.

Para consultar uma transação basta chamar função `SuperpayApi::Transacao.consulta_transacao(numero_transacao)`, será retornado um objeto do tipo `SuperpayApi::Retorno`. \(Verifique o objeto mais abaixo\). Tal objeto possui todas as informações necessárias para validar o resultado da transação.

```ruby
retorno = SuperpayApi::Transacao.consulta_transacao(numero_transacao)
```

## Capturar transação

Em algumas operadoras e instituições financeiras, é possível realizar a aprovação de uma transação em duas etapas, chamadas **autorização** e **captura**.

### Autorização

A etapa inicial do processo, onde a operadora financeira é acionada pelo SuperPay. Essa etapa verifica a condição de crédito do cliente, ou seja, verifica se o mesmo possui crédito suficiente para realizar a compra. Em casos positivos, aquele valor é reservado na conta do cliente para que o processo de captura ocorra.

É preciso configurar o meio de pagamento como captura manual na conta SuperPay, assim quando enviar a transação pelo método **Criar transação** `transacao.enviar_pagamento` o pedido será apenas autorizado, nesse caso o próprio estabelecimento decide o momento de realizar a **Captura**.

### Captura

A confirmação da transação. Nesta etapa o SuperPay aciona a operadora financeira para confirmar uma transação previamente autorizada. Somente nessa etapa é que é realizada a cobrança do cliente.

O SuperPay oferece tanto o sistema de captura automática quanto o de captura manual, onde o próprio estabelecimento decide o momento de realizar a etapa e também é o responsável por acionar o processo.

Para capturar uma transação basta chamar função `SuperpayApi::Transacao.capturar_transacao(numero_transacao)`, será retornado um objeto do tipo `SuperpayApi::Retorno`. \(Verifique o objeto mais abaixo\). Tal objeto possui todas as informações necessárias para validar o resultado da transação.

```ruby
retorno = SuperpayApi::Transacao.capturar_transacao(numero_transacao)
```

## Cancelar transação

Outra funcionalidade disponível no SuperPay é o cancelamento de transações de acordo com a disponibilidade do serviço nas operadoras e instituições financeiras disponível nesse [link](http://wiki.superpay.com.br/wikiSuperPay/index.php/Cancelar_transa%C3%A7%C3%A3o_SOAP#Observa.C3.A7.C3.B5es_sobre_o_Cancelamento_da_Transa.C3.A7.C3.A3o).

Para cancelar uma transação basta chamar função `SuperpayApi::Transacao.cancelar_transacao(numero_transacao)`, será retornado um objeto do tipo `SuperpayApi::Retorno`. \(Verifique o objeto mais abaixo\). Tal objeto possui todas as informações necessárias para validar o resultado da transação.

```ruby
retorno = SuperpayApi::Transacao.cancelar_transacao(numero_transacao)
```

## Estorno de transação

Funcionalidade para estornar o valor do pedido, tanto **parcial** quanto **total**, para o cliente.

_**Item disponível somente para clientes pós-pago e que utilizam operadora Cielo. Para maiores detalhes entrar em contato com o Comercial \(comercial@superpay.com.br\).**_

Para estornar uma transação basta chamar função `SuperpayApi::Transacao.estorno_de_transacao(numero_transacao, valor_estorno)`, será retornado uma mensagem informando o cadastro do estorno.

Para saber se o estorno foi ou não realizado pela operadora, deve-se realizar **Consultar transação** específica ao pedido.

**Tempo de resposta de até 30 minutos.**

```ruby
# Sempre enviar o valor sem vírgula ou ponto, os dois últimos dígitos são sempre considerados como centavos
retorno = SuperpayApi::Transacao.estorno_de_transacao(numero_transacao, valor_estorno)
```

## Campainha

O sistema de campainha existe para notificar o estabelecimento sobre uma atualização de status na transação. Toda vez que ocorre qualquer alteração de status em uma transação, é feita uma chamada via POST ao campo `:url_campainha` (cadastrado nas configurações).

Para localizar e consultar uma transação a partir da campainha basta chamar função `SuperpayApi::Transacao.localizar_pela_campainha(notificacao)`, será retornado um objeto do tipo `SuperpayApi::Retorno`. \(Verifique o objeto mais abaixo\). Tal objeto possui todas as informações necessárias para validar o resultado da transação.

```ruby
# notificacao => Hash com os parâmetros do POST
retorno = SuperpayApi::Transacao.localizar_pela_campainha(notificacao)
```

## Exemplo de [Retorno](http://www.rubydoc.info/github/qw3/superpay_api/SuperpayApi/Retorno)

```ruby
retorno = {
  :numero_transacao             => '1',
  :codigo_estabelecimento       => '1010101010101010',
  :codigo_forma_pagamento       => '120',
  :valor                        => '28260',
  :valor_desconto               => '0',
  :taxa_embarque                => '0',
  :parcelas                     => '1',
  :url_pagamento                => '14132971582229c00506d-e84d-4526-b902-92190d5aa808',
  :status_transacao             => '1',
  :autorizacao                  => '******',
  :codigo_transacao_operadora   => '0',
  :data_aprovacao_operadora     => '20/01/2016',
  :numero_comprovante_venda     => '1006993069181F841001',
  :mensagem_venda               => 'Transacao capturada com sucesso',
}
```

### Exemplo de [Retorno com Erro](http://www.rubydoc.info/github/qw3/superpay_api/SuperpayApi/Retorno)

Sempre verifique se o objeto retornardo tem errors: `retorno.errors.blank?`

```ruby
retorno = SuperpayApi::Transacao.consulta_transacao -1

retorno.errors.blank?
# => false

retorno.errors.full_messages
# => ["Code soap:Server", "Mensagem Problemas ao consultar transação. Transação não encontrada"]
```

## Documentação

- [RubyDoc](http://www.rubydoc.info/github/qw3/superpay_api)
- [RubyGems](https://rubygems.org/gems/superpay_api)
- [GitHub](https://github.com/qw3/superpay_api)


---

## Contributing

- Check out the latest master to make sure the feature hasn't been implemented or the bug hasn't been fixed yet.
- Check out the issue tracker to make sure someone already hasn't requested it and/or contributed it.
- Fork the project.
- Start a feature/bugfix branch.
- Commit and push until you are happy with your contribution.
- Don't forget to rebase with branch master in main project before submit the pull request.
- Please don't change the gemspec file, Rakefile, version, or history.

## Autor
- [Leandro Falcão](https://github.com/lsfalcao)

## Copyright

- [Leandro Falcão](https://github.com/lsfalcao)
- [QW3 Software & Marketing](https://qw3.com.br)

![QW3 Logo](http://qw3.com.br/qw3_logo.png)

The MIT License (MIT)

Copyright (c) 2019 QW3

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
