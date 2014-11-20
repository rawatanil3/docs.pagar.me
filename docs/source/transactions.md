---
title: Pagamento dentro do seu site

language_tabs:
  - shell
  - ruby
  - php

search: true
---

# Realizando uma transação

Após [capturar os dados de cartão, gerar o `card_hash` e enviá-lo para o seu
servidor](/capturing-card-data), você deve realizar a transação junto ao
Pagar.me, que efetuará a transação no cartão do cliente.

Para transações de boleto bancário, a transação pode ser realizada totalmente
dentro do seu servidor, já que não há a transmissão de dados sensíveis e
portanto o `card_hash` não é utilizado.

## Realizando uma transação de cartão de crédito {#credit-card-transaction}

Com o `card_hash` em mãos no seu servidor, agora basta realizar a transação:

```shell
curl -X POST 'https://api.pagar.me/1/transactions' \
    -d 'api_key=ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0' \
    -d 'amount=1000' \
    -d 'card_hash={CARD_HASH}'
```

```ruby
require 'pagarme'

PagarMe.api_key = "ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0";

transaction = PagarMe::Transaction.new({
	:amount => 1000,
    :card_hash => "{CARD_HASH}"
})

transaction.charge

status = transaction.status # status da transação
```

```php
<?php
	require("pagarme-php/Pagarme.php");

	Pagarme::setApiKey("ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0");

	$transaction = new PagarMe_Transaction(array(
		'amount' => 1000,
		'card_hash' => "{CARD_HASH}"
	));

	$transaction->charge();

	$status = $transaction->status; // status da transação
?>
```

> Não se esqueça de substituir `ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0` pela
> sua chave de API disponível no seu [Dashboard](https://dashboard.pagar.me/).

Após realizar uma transação de cartão de crédito, a mesma terá status
`paid`, indicando que o cartão do usuário foi debitado com sucesso.

Se a transação for recusada pelas operadoras de cartão, a mesma terá status
`refused`.

### Configurações da transação

Tag | Padrão | Descrição
--- | ------ | ---------
amount | --- | Valor total a ser cobrado (em centavos). Ex: R$14,99 = `1499`
card_hash | --- | Representação segura dos dados de cartão de crédito gerada pelo `Pagarme.js`
installments | `1` | Número de parcelas a ser cobrado no cartão de crédito.
payment_method | `credit_card` | Meio de pagamento a ser utilizado
postback_url | --- | URL para receber notificações sobre alterações da transação ([saiba mais](#postback))
soft_descriptor | --- | Texto (de até 13 caracteres) que será aparecerá na fatura do cartão do cliente ao lado do nome da sua loja
capture | `true` | Passar `false` para apenas autorizar a transação e realizar a captura posteriormente

## Realizando uma transação de boleto bancário {#boleto-transaction}

Uma transação de boleto bancária deve ser realizada diretamente do seu
servidor, sem a necessidade de utilizar o `card_hash`, já que não há
transmissão de dados sensíveis para o seu servidor e para o Pagar.me.

```shell
curl -X POST 'https://api.pagar.me/1/transactions' \
    -d 'api_key=ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0' \
    -d 'amount=1000' \
    -d 'payment_method=boleto'
```

```ruby
require 'pagarme'

PagarMe.api_key = "ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0";

transaction = PagarMe::Transaction.new({
	:amount => 1000,
    :payment_method => "boleto"
})

transaction.charge

boleto_url = transaction.boleto_url # URL do boleto bancário
boleto_barcode = transaction.boleto_barcode # código de barras do boleto bancário
```

```php
<?php
	require("pagarme-php/Pagarme.php");

	Pagarme::setApiKey("ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0");

	$transaction = new PagarMe_Transaction(array(
		'amount' => 1000,
		'payment_method' => "boleto"
	));

	$transaction->charge();

	$boleto_url = $transaction->boleto_url; // URL do boleto bancário
	$boleto_barcode = $transaction->boleto_barcode; // código de barras do boleto bancário
?>
```

> Não se esqueça de substituir `ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0` pela
> sua chave de API disponível no seu [Dashboard](https://dashboard.pagar.me/).

A transação terá status `waiting_payment` até o pagamento do boleto
bancário. A URL do boleto bancário para pagamento estará disponível na variável
`boleto_url`.

Quando o boleto bancário for detectado como pago, a transação passará a ter o
status `paid`.

### Configurações da transação

Tag | Padrão | Descrição
--- | ------ | ---------
amount | --- | Valor total a ser cobrado (em centavos). Ex: R$14,99 = `1499`
payment_method | `boleto` | Meio de pagamento a ser utilizado
postback_url | --- | URL para receber notificações sobre alterações da transação ([saiba mais](#postback))
boleto_expiration_date | data atual + 7 dias | Data de vencimento do boleto bancário

## Recebendo notificações de mudança de status da transação (POSTback) {#postback}

Ao realizar uma transação, diversos serviços externos (como empresas de
antifraude, operadoras de cartão e bancos) são consultados. Por isso, esse
processo pode demorar um pouco mais do que uma requisição HTTP normal, que
costuma durar menos de um segundo.

Para evitar que o tempo de resposta da requisição prejudique sua aplicação, ao
criar uma transação, você pode fornecer uma `postback_url`. Dessa forma, quando
a transação mudar de status, o Pagar.me notificará sua aplicação e o fluxo de
compra pode ser finalizado.

As URLs de POSTback também são úteis ao criar transações de boleto bancário.
Dessa forma, quando o pagamento do boleto for detectado junto ao banco, sua
aplicação será notificada e o fluxo de compra pode ser finalizado.

Caso a transação seja estornada, a URL de POSTback também será notificada sobre
a mudança de status.

```shell
curl -X POST 'https://api.pagar.me/1/transactions' \
    -d 'api_key=ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0' \
    -d 'amount=1000' \
    -d 'card_hash={CARD_HASH}' \
	-d 'postback_url=http://seusite.com/payments/2718'
```

```ruby
require 'pagarme'

PagarMe.api_key = "ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0";

transaction = PagarMe::Transaction.new({
	:amount => 1000,
    :card_hash => "{CARD_HASH}",
	:postback_url => "http://seusite.com/payments/2718"
})

transaction.charge
```

```php
<?php
	require("pagarme-php/Pagarme.php");

	Pagarme::setApiKey("ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0");

	$transaction = new PagarMe_Transaction(array(
		'amount' => 1000,
		'card_hash' => "{CARD_HASH}",
		'postback_url' => "http://seusite.com/payments/2718"
	));

	$transaction->charge();
?>
```

> Não se esqueça de substituir `ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0` pela
> sua chave de API disponível no seu [Dashboard](https://dashboard.pagar.me/).

<aside class="notice">Você pode utilizar o serviço
[RequestBin](http://requestb.in) para gerar URLs de POSTback de teste e
visualizar as requisições enviadas pelo Pagar.me.</aside>

Quando a `postback_url` é passada, a transação será retornada com status
`processing`, e as mudanças de status serão enviadas para o seu servidor na URL
de POSTback, através de um request `HTTP POST` com os seguintes parâmetros:

Parâmetro | Descrição | Valor padrão 
--- | --- | ---------
object | Objeto que originou a notificação de POSTback | `transaction`
id | ID do objeto (transação) que originou a notificação de POSTback | ---
event | Evento que originou a notificação de POSTback | `transaction_status_changed`
current_status | Status da transação após o evento | ---
old_status | Status da transação antes do evento | ---
desired_status | Status desejado após o evento | Ao criar a transação: `paid`. Ao estornar a transação: `refunded` |
fingerprint | Parâmetro usado para validar a notificação de POSTback ([saiba mais](/advanced#validando-a-origem-de-um-postback)) | ---

## Enviando dados do cliente para o Pagar.me (antifraude) {#customer-data}

Para ter um maior controle dos seus clientes, principalmente através do
Dashboard do Pagar.me, recomendamos que você também envie os dados do cliente
ao realizar a transação.

<aside class="notice">Para utilizar o recurso antifraude, é obrigatório o envio
dos dados do cliente para o Pagar.me.</aside>

```shell
curl -X POST 'https://api.pagar.me/1/transactions' \
    -d 'api_key=ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0' \
    -d 'amount=1000' \
    -d 'card_hash={CARD_HASH}' \
	-d 'customer[name]=John Appleseed' \
	-d 'customer[document_number]=92545278157' \
	-d 'customer[email]=jappleseed@apple.com' \
	-d 'customer[address][street]=Av. Brigadeiro Faria Lima' \
	-d 'customer[address][neighborhood]=Jardim Paulistano' \
	-d 'customer[address][zipcode]=01452000' \
	-d 'customer[address][street_number]=2941' \
	-d 'customer[address][complementary]=8º andar' \
	-d 'customer[phone][ddd]=11' \
	-d 'customer[phone][number]=30713261' \
```

```ruby
require 'pagarme'

PagarMe.api_key = "ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0";

transaction = PagarMe::Transaction.new({
	:amount => 1000,
    :card_hash => "{CARD_HASH}",
	:customer => {
		:name => "John Appleseed",
		:document_number => "92545278157",
		:email => "jappleseed@apple.com",
		:address => {
			:street => "Av. Brigadeiro Faria Lima",
			:neighborhood => "Jardim Paulistano",
			:zipcode => "01452000",
			:street_number => "2941",
			:complementary => "8º andar"
		},
		:phone => {
			:ddd => "11"
			:number => "30713261"
		}
	}
})

transaction.charge
```

```php
<?php
	require("pagarme-php/Pagarme.php");

	Pagarme::setApiKey("ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0");

	$transaction = new PagarMe_Transaction(array(
		"amount" => 1000,
		"card_hash" => "{CARD_HASH}",
		"customer" => array(
			"name" => "John Appleseed",
			"document_number" => "92545278157",
			"email" => "jappleseed@apple.com",
			"address" => array(
				"street" => "Av. Brigadeiro Faria Lima",
				"neighborhood" => "Jardim Paulistano",
				"zipcode" => "01452000",
				"street_number" => "2941",
				"complementary" => "8º andar"
			),
			"phone" => array(
				"ddd" => "11",
				"number" => "30713261"
			)
		}
	));

	$transaction->charge();
?>
```
> Não se esqueça de substituir `ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0` pela
> sua chave de API disponível no seu [Dashboard](https://dashboard.pagar.me/).