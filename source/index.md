---
title: Manual Checkout Cielo

language_tabs:
  - json: JSON
  - shell: Shell
  - php: PHP
  - ruby: Ruby
  - python: Python
  - java: Java
  - csharp: C#

toc_footers:
  - <a href='/Checkout-Backoffice/'>Backoffice Cielo</a>
  - <a href='/Checkout-FAQ/'>Perguntas Frequêntes</a>

search: true
---

# Manual Checkout Cielo

O objetivo desta documentação é orientar o desenvolvedor sobre o método de integração da API Checkout Cielo, solução simplificada na qual o consumidor é direcionado para uma página de pagamento online segura da Cielo, proporcionando um alto nível de confiança, dentro das mais rígidas normas de segurança (PCI). Em linhas gerais, o Checkout Cielo é uma solução de pagamento projetada para aumentar a conversão das vendas, simplificar o processo de compra, reduzir fraudes e custos operacionais.
Nesta documentação estão descritas todas as funcionalidades desta integração, os parâmetros técnicos e principalmente os códigos de exemplos para facilitar o seu desenvolvimento. 

O Checkout Cielo utiliza uma tecnologia REST que deve ser usada quando houver um *“carrinho de compras”* a ser enviado, ou seja, no caso do consumidor navegar pelo site e escolher 1 ou mais produtos para adicionar ao carrinho e depois, então, finalizar a compra. Há também opção de integração via botão usada sempre que não houver um *“carrinho de compras”* em sua loja ou quando se deseja associar uma compra rápida direta a um produto.

Durante a integração com o Checkout Cielo, alguns passos e alguns redirecionamentos ocorrerão. A imagem abaixo ilustra esse fluxo:

![Fluxo de integração Checkout Cielo](/images/fluxo-checkout.svg)

Após o portador do cartão (consumidor) selecionar suas compras e apertar o botão “Comprar” da loja virtual já integrada ao Checkout Cielo, o fluxo segue desta forma:

1. A API da Cielo retorna o **CheckoutURL**, que deverá ser utilizado pela loja para redirecionar o cliente ao ambiente seguro de pagamento da Cielo (página do Checkout Cielo)
2. A loja redireciona o cliente para a URL retornada pela Cielo
3. O portador do cartão digita os dados de pagamento e conclui a compra
4. O Checkout Cielo redireciona o cliente para a URL de Retorno escolhida pela loja, configurada no [Backoffice Checkout Cielo](/Checkout-Backoffice/) desta solução
5. A loja avisa ao cliente que o processo foi concluído e que ele receberá mais informações sobre a compra e o pagamento por e-mail.
6. O Checkout Cielo envia o POST de notificação para a URL de Notificação, configurada no Backoffice
7. A loja processa o pedido de compra utilizando os dados do POST de notificação e, se a transação estiver autorizada, libera o pedido.

# Visão Geral

Neste manual será apresentada uma visão geral do Checkout Cielo e o mecanismo tecnológico da integração com carrinho ou com botão. Para todo pedido de compra, a meta é revertê-lo em uma venda. Uma venda com cartão pode ser caracterizada por uma transação autorizada e capturada.

<aside class="warning">Uma transação autorizada somente gera o crédito para o lojista se ela for capturada (ou confirmada).</aside>

Após a conclusão da etapa de integração com o Checkout Cielo, é fundamental que o lojista ou administrador da loja online tenha conhecimento dos processos funcionais que farão parte do cotidiano da loja, como o acompanhamento das movimentações financeiras, status de cada venda, tomada de ações  (captura e cancelamento) com relação às vendas,  extrato de cobrança, entre outros. Veja o material complementar sobre o [BackOffice Checkout Cielo](http://developercielo.github.io/Checkout-Backoffice/). 


## Considerações sobre a integração

* O cadastro da loja deve estar ativo junto à Cielo.
* Deve-se definir um timeout adequado nas requisições HTTP à Cielo; recomendamos 30 segundos.
* O certificado Root da entidade certificadora (CA) de nosso Web Service deve estar cadastrado na Truststore a ser utilizada. Como nossa certificadora é de ampla aceitação no mercado, é provável que ela já esteja registrada na Truststore do próprio sistema operacional. Veja a seção [Certificado Extended Validation](#certificado-extended-validation) para mais informações.
* Os valores monetários são sempre tratados como valores inteiros, sem representação das casas decimais, sendo que os dois últimos dígitos são considerados como os centavos. Exemplo: R$ 1.286,87 é representado como 128687; R$ 1,00 é representado como 100.

<aside class="notice">Veja a seção <a href="#certificado-extended-validation">Certificado Extended Validation</a> para informações sobre os certificados Cielo</aside>

## Produtos e serviços

A versão atual do Checkout Cielo possui suporte às seguintes bandeiras e produtos:

|Bandeira|Crédito à vista|Crédito parcelado Loja|Débito|Voucher|
|--------|---------------|----------------------|------|-------|
|Visa|Sim|Sim|Sim|Não|
|Master Card|Sim|Sim|Sim|Não|
|American Express|Sim|Sim|Não|Não|
|Elo|Sim|Sim|Não|Não|
|Diners Club|Sim|Sim|Não|Não|
|Discover|Sim|Não|Não|Não|
|JCB|Sim|Sim|Não|Não|
|Aura|Sim|Sim|Não|Não|

## Histórico de versões

* **Versão 1.3** - 21/01/2015
    - Troca de nomes – Solução Integrada para Checkout Cielo
* **Versão 1.2** - 09/01/2015
    - Inclusão dos seguintes parâmetros no Post de notificação: `discount_amount`, `shipping_address_state`, `payment_boleto`, `number`, `tid`;
    - Alteração do parâmetro numero do pedido no Post de Mudança de Status
* **Versão 1.1** - 08/01/2015
    - Alinhamento dos fluxos de pagamento; inclusão de informações sobre os meios de pagamento; inclusão da tela de configurações do [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/)
* **Versão 1.0** - 24/11/2014
    - Versão inicial

## Suporte Cielo

Após a leitura deste manual, caso ainda persistam dúvidas (técnicas ou não), a Cielo disponibiliza o suporte técnico 24 horas por dia, 7 dias por semana em idiomas (Português e Inglês), nos seguintes contatos:

* +55 4002-9700 – *Capitais e Regiões Metropolitanas*
* +55 0800-570-1700 – *Demais Localidades*
* +55 11 2860-1348 – *Internacionais*
  * Opção 1 – *Suporte técnico;*
  * Opção 2 – *Credenciamento E-commerce.*
* Email: [cieloecommerce@cielo.com.br](mailto:cieloecommerce@cielo.com.br)

# Integração

## Carrinho de Compras

Este  tipo  de  integração deve  ser  usada sempre  que  houver  um  “carrinho  de  compras”  a  ser  enviado,  ou  seja,  no  caso  do consumidor navegar pelo site e escolher 1 ou mais produtos para adicionar a um carrinho e depois então finalizar a venda. Se você não possui um carrinho de compras implementado, veja a seção de [integração via botão](#integração-via-botão) Checkout Cielo.

### Endpoint

Endpoint é a URL para onde as requisições com os dados do carrinho serão enviadas. Todas as requisições deverão ser enviadas utilizando o método HTTP POST, para o endpoint `https://cieloecommerce.cielo.com.br/api/public/v1/orders`.

### Autenticação da loja

```shell
-H "MerchantId: 00000000-0000-0000-0000-000000000000" \
```

```php
<?php
curl_setopt($curl, CURLOPT_HTTPHEADER, array('MerchantId: 00000000-0000-0000-0000-000000000000'));
```

```ruby
headers  = {:content_type => "application/json",:merchantid => "00000000-0000-0000-0000-000000000000"}
```

```python
headers = {"Content-Type": "application/json", "MerchantId": "00000000-0000-0000-0000-000000000000"}
```

```java
connection.addRequestProperty("MerchantId", "0000000-0000-0000-0000-000000000000");
```

```csharp
request.Headers["MerchantId"] = "00000000-0000-0000-0000-000000000000";
```

Por se tratar de transações, todas as requisições enviadas para a Cielo deverão ser autenticadas pela loja. A autenticação consiste no envio do Merchant Id, que é o identificador único da loja fornecido pela Cielo após a afiliação da loja. A autenticação da loja deverá ser feita através do envio do campo de cabeçalho HTTP `MerchandId`, como ilustrado abaixo e ao lado:

`MerchantId: 00000000-0000-0000-0000-000000000000`

<aside class="notice">
Lembre-se de substituir `00000000-0000-0000-0000-000000000000` pelo seu MerchantId.
</aside>

### Requisição

```json
{
    "OrderNumber": "12344",
    "SoftDescriptor": "Nome que aparecerá na fatura",
    "Cart": {
        "Discount": {
            "Type": "Percent",
            "Value": 10
        },
        "Items": [
            {
                "Name": "Nome do produto",
                "Description": "Descrição do produto",
                "UnitPrice": 100,
                "Quantity": 2,
                "Type": "Asset",
                "Sku": "Sku do item no carrinho",
                "Weight": 200
            }
        ]
    },
    "Shipping": {
        "Type": "Correios",
        "SourceZipCode": "14400000",
        "TargetZipCode": "11000000",
        "Address": {
            "Street": "Endereço de entrega",
            "Number": "123",
            "Complement": "",
            "District": "Bairro da entrega",
            "City": "Cidade de entrega",
            "State": "São Paulo"
        },
        "Services": [
            {
                "Name": "Serviço de frete",
                "Price": 123,
                "Deadline": 15
            }
        ]
    },
    "Payment": {
        "BoletoDiscount": 0,
        "DebitDiscount": 10
    },
    "Customer": {
        "Identity": 11111111111,
        "FullName": "Fulano Comprador da Silva",
        "Email": "fulano@email.com",
        "Phone": "11999999999"
    },
    "Options": {
        "AntifraudEnabled": false
    }
}
```

```shell
curl -X POST "https://cieloecommerce.cielo.com.br/api/public/v1/orders" \
     -H "MerchantId: 00000000-0000-0000-0000-000000000000" \
     -H "Content-Type: application/json" \
     -d '{
             "OrderNumber": "12344",
             "SoftDescriptor": "Nome que aparecerá na fatura",
             "Cart": {
                  "Discount": {
                      "Type": "Percent",
                      "Value": 10
                  },
                  "Items": [
                      {
                          "Name": "Nome do produto",
                          "Description": "Descrição do produto",
                          "UnitPrice": 100,
                          "Quantity": 2,
                          "Type": "Asset",
                          "Sku": "Sku do item no carrinho",
                          "Weight": 200
                      }
                  ]
             },
             "Shipping": {
                  "Type": "Correios",
                  "SourceZipCode": "14400000",
                  "TargetZipCode": "11000000",
                  "Address": {
                      "Street": "Endereço de entrega",
                      "Number": "123",
                      "Complement": "",
                      "District": "Bairro da entrega",
                      "City": "Cidade de entrega",
                      "State": "São Paulo"
                  },
                  "Services": [
                      {
                          "Name": "Serviço de frete",
                          "Price": 123,
                          "Deadline": 15
                      }
                  ]
             },
             "Payment": {
                  "BoletoDiscount": 0,
                  "DebitDiscount": 10
             },
             "Customer": {
                  "Identity": 11111111111,
                  "FullName": "Fulano Comprador da Silva",
                  "Email": "fulano@email.com",
                  "Phone": "11999999999"
             },
             "Options": {
                  "AntifraudEnabled": false
             }
         }'
```

```php
<?php
$order = new stdClass();
$order->OrderNumber = '1234';
$order->SoftDescriptor = 'Nome que aparecerá na fatura';
$order->Cart = new stdClass();
$order->Cart->Discount = new stdClass();
$order->Cart->Discount->Type = 'Percent';
$order->Cart->Discount->Value = 10;
$order->Cart->Items = array();
$order->Cart->Items[0] = new stdClass();
$order->Cart->Items[0]->Name = 'Nome do produto';
$order->Cart->Items[0]->Description = 'Descrição do produto';
$order->Cart->Items[0]->UnitPrice = 100;
$order->Cart->Items[0]->Quantity = 2;
$order->Cart->Items[0]->Type = 'Asset';
$order->Cart->Items[0]->Sku = 'Sku do item no carrinho';
$order->Cart->Items[0]->Weight = 200;
$order->Shipping = new stdClass();
$order->Shipping->Type = 'Correios';
$order->Shipping->SourceZipCode = '14400000';
$order->Shipping->TargetZipCode = '11000000';
$order->Shipping->Address = new stdClass();
$order->Shipping->Address->Street = 'Endereço de entrega';
$order->Shipping->Address->Number = '123';
$order->Shipping->Address->Complement = '';
$order->Shipping->Address->District = 'Bairro da entrega';
$order->Shipping->Address->City = 'Cidade da entrega';
$order->Shipping->Address->State = 'São Paulo';
$order->Shipping->Services = array();
$order->Shipping->Services[0] = new stdClass();
$order->Shipping->Services[0]->Name = 'Serviço de frete';
$order->Shipping->Services[0]->Price = 123;
$order->Shipping->Services[0]->DeadLine = 15;
$order->Payment = new stdClass();
$order->Payment->BoletoDiscount = 0;
$order->Payment->DebitDiscount = 10;
$order->Customer = new stdClass();
$order->Customer->Identity = 11111111111;
$order->Customer->FullName = 'Fulano Comprador da Silva';
$order->Customer->Email = 'fulano@email.com';
$order->Customer->Phone = '11999999999';
$order->Options = new stdClass();
$order->Options->AntifraudEnabled = false;

$curl = curl_init();

curl_setopt($curl, CURLOPT_URL, 'https://cieloecommerce.cielo.com.br/api/public/v1/orders');
curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, json_encode($order));
curl_setopt($curl, CURLOPT_HTTPHEADER, array(
    'MerchantId: 00000000-0000-0000-0000-000000000000',
    'Content-Type: application/json'
));

$response = curl_exec($curl);

curl_close($curl);

$json = json_decode($response);
```

```python
from urllib2 import Request, urlopen
from json import dumps

json = dumps({
    "OrderNumber": "12344",
    "SoftDescriptor": "Nome que aparecerá na fatura",
    "Cart": {
        "Discount": {
            "Type": "Percent",
            "Value": 10
        },
        "Items": [
            {
                "Name": "Nome do produto",
                "Description": "Descrição do produto",
                "UnitPrice": 100,
                "Quantity": 2,
                "Type": "Asset",
                "Sku": "Sku do item no carrinho",
                "Weight": 200
            }
        ]
    },
    "Shipping": {
        "Type": "Correios",
        "SourceZipCode": "14400000",
        "TargetZipCode": "11000000",
        "Address": {
            "Street": "Endereço de entrega",
            "Number": "123",
            "Complement": "",
            "District": "Bairro da entrega",
            "City": "Cidade de entrega",
            "State": "São Paulo"
        },
        "Services": [
            {
                "Name": "Serviço de frete",
                "Price": 123,
                "Deadline": 15
            }
        ]
    },
    "Payment": {
        "BoletoDiscount": 0,
        "DebitDiscount": 10
    },
    "Customer": {
        "Identity": 11111111111,
        "FullName": "Fulano Comprador da Silva",
        "Email": "fulano@email.com",
        "Phone": "11999999999"
    },
    "Options": {
        "AntifraudEnabled": false
    }
})

headers = {"Content-Type": "application/json", "MerchantId": "00000000-0000-0000-0000-000000000000"}
request = Request("https://cieloecommerce.cielo.com.br/api/public/v1/orders", data=json, headers=headers)
response = urlopen(request).read()

print response
```

```ruby
require 'rubygems' if RUBY_VERSION < '1.9'
require 'rest-client'
require 'json'

request = JSON.generate({
    "OrderNumber" => "12344",
    "SoftDescriptor" => "Nome que aparecerá na fatura",
    "Cart" => {
        "Discount" => {
            "Type" => "Percent",
            "Value" => 10
        },
        "Items" => [
            {
                "Name" => "Nome do produto",
                "Description" => "Descrição do produto",
                "UnitPrice" => 100,
                "Quantity" => 2,
                "Type" => "Asset",
                "Sku" => "Sku do item no carrinho",
                "Weight" => 200
            }
        ]
    },
    "Shipping" => {
        "Type" => "Correios",
        "SourceZipCode" => "14400000",
        "TargetZipCode" => "11000000",
        "Address" => {
            "Street" => "Endereço de entrega",
            "Number" => "123",
            "Complement" => "",
            "District" => "Bairro da entrega",
            "City" => "Cidade de entrega",
            "State" => "São Paulo"
        },
        "Services" => [
            {
                "Name" => "Serviço de frete",
                "Price" => 123,
                "Deadline" => 15
            }
        ]
    },
    "Payment" => {
        "BoletoDiscount" => 0,
        "DebitDiscount" => 10
    },
    "Customer" => {
        "Identity" => 11111111111,
        "FullName" => "Fulano Comprador da Silva",
        "Email" => "fulano@email.com",
        "Phone" => "11999999999"
    },
    "Options" => {
        "AntifraudEnabled" => false
    }
})

headers  = {:content_type => "application/json",:merchantid => "00000000-0000-0000-0000-000000000000"}
response = RestClient.post "https://cieloecommerce.cielo.com.br/api/public/v1/orders", request, headers

puts response
```

```java
String json = "{"
            + "    \"OrderNumber\": \"12344\","
            + "    \"SoftDescriptor\": \"Nome que aparecerá na fatura\","
            + "    \"Cart\": {"
            + "        \"Discount\": {"
            + "            \"Type\": \"Percent\","
            + "            \"Value\": 10"
            + "        },"
            + "        \"Items\": ["
            + "            {"
            + "                \"Name\": \"Nome do produto\","
            + "                \"Description\": \"Descrição do produto\","
            + "                \"UnitPrice\": 100,"
            + "                \"Quantity\": 2,"
            + "                \"Type\": \"Asset\","
            + "                \"Sku\": \"Sku do item no carrinho\","
            + "                \"Weight\": 200"
            + "            }"
            + "        ]"
            + "    },"
            + "    \"Shipping\": {"
            + "        \"Type\": \"Correios\","
            + "        \"SourceZipCode\": \"14400000\","
            + "        \"TargetZipCode\": \"11000000\","
            + "        \"Address\": {"
            + "            \"Street\": \"Endereço de entrega\","
            + "            \"Number\": \"123\","
            + "            \"Complement\": \"\","
            + "            \"District\": \"Bairro da entrega\","
            + "            \"City\": \"Cidade de entrega\","
            + "            \"State\": \"São Paulo\""
            + "        },"
            + "        \"Services\": ["
            + "            {"
            + "                \"Name\": \"Serviço de frete\","
            + "                \"Price\": 123,"
            + "                \"Deadline\": 15"
            + "            }"
            + "        ]"
            + "    },"
            + "    \"Payment\": {"
            + "        \"BoletoDiscount\": 0,"
            + "        \"DebitDiscount\": 10"
            + "     },"
            + "     \"Customer\": {"
            + "         \"Identity\": 11111111111,"
            + "         \"FullName\": \"Fulano Comprador da Silva\","
            + "         \"Email\": \"fulano@email.com\","
            + "         \"Phone\": \"11999999999\""
            + "     },"
            + "     \"Options\": {"
            + "         \"AntifraudEnabled\": false"
            + "     }"
            + "}";

URL url;
HttpURLConnection connection;
BufferedReader bufferedReader;

try {
    url = new URL("https://cieloecommerce.cielo.com.br/api/public/v1/orders");

    connection = (HttpURLConnection) url.openConnection();
    connection.setRequestMethod("POST");
    connection.addRequestProperty("MerchantId", "0000000-0000-0000-0000-000000000000");
    connection.setRequestProperty("Content-Type", "application/json; charset=utf-8");
    connection.setDoOutput(true);

    DataOutputStream jsonRequest = new DataOutputStream(
                connection.getOutputStream());

    jsonRequest.writeBytes(json);
    jsonRequest.flush();
    jsonRequest.close();

    bufferedReader = new BufferedReader(new InputStreamReader(
                connection.getInputStream()));

    String responseLine;
    StringBuffer jsonResponse = new StringBuffer();

    while ((responseLine = bufferedReader.readLine()) != null) {
        jsonResponse.append(responseLine);
    }

    bufferedReader.close();

    connection.disconnect();
} catch (Exception e) {
    e.printStackTrace();
}
```

```csharp
HttpWebRequest request = (HttpWebRequest)
                         WebRequest.Create("https://cieloecommerce.cielo.com.br/api/public/v1/orders");

request.Method = "POST";
request.Headers["Content-Type"] = "text/json";
request.Headers["MerchantId"] = "06eadc0b-2e32-449b-be61-6fd4f1811708";

string json = "{"
            + "    \"OrderNumber\": \"12344\","
            + "    \"SoftDescriptor\": \"Nome que aparecerá na fatura\","
            + "    \"Cart\": {"
            + "        \"Discount\": {"
            + "            \"Type\": \"Percent\","
            + "            \"Value\": 10"
            + "        },"
            + "        \"Items\": ["
            + "            {"
            + "                \"Name\": \"Nome do produto\","
            + "                \"Description\": \"Descrição do produto\","
            + "                \"UnitPrice\": 100,"
            + "                \"Quantity\": 2,"
            + "                \"Type\": \"Asset\","
            + "                \"Sku\": \"Sku do item no carrinho\","
            + "                \"Weight\": 200"
            + "            }"
            + "        ]"
            + "    },"
            + "    \"Shipping\": {"
            + "        \"Type\": \"Correios\","
            + "        \"SourceZipCode\": \"14400000\","
            + "        \"TargetZipCode\": \"11000000\","
            + "        \"Address\": {"
            + "            \"Street\": \"Endereço de entrega\","
            + "            \"Number\": \"123\","
            + "            \"Complement\": \"\","
            + "            \"District\": \"Bairro da entrega\","
            + "            \"City\": \"Cidade de entrega\","
            + "            \"State\": \"São Paulo\""
            + "        },"
            + "        \"Services\": ["
            + "            {"
            + "                \"Name\": \"Serviço de frete\","
            + "                \"Price\": 123,"
            + "                \"Deadline\": 15"
            + "            }"
            + "        ]"
            + "    },"
            + "    \"Payment\": {"
            + "        \"BoletoDiscount\": 0,"
            + "        \"DebitDiscount\": 10"
            + "     },"
            + "     \"Customer\": {"
            + "         \"Identity\": 11111111111,"
            + "         \"FullName\": \"Fulano Comprador da Silva\","
            + "         \"Email\": \"fulano@email.com\","
            + "         \"Phone\": \"11999999999\""
            + "     },"
            + "     \"Options\": {"
            + "         \"AntifraudEnabled\": false"
            + "     }"
            + "}";

using (var writer = new StreamWriter(request.GetRequestStream()))
{
    writer.Write(json);
	writer.Close();
}

HttpWebResponse response = (HttpWebResponse) request.GetResponse();
```

#### Cabeçalho HTTP

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|MerchantId|Guid|Sim|36|Identificador único da loja. **Formato:** 00000000-0000-0000-0000-000000000000|
|Content-type|Alphanumeric|Sim|n/a|Tipo do conteúdo da mensagem a ser enviada. **Utilizar:** "application/json"|

#### Objeto raiz da requisição

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|OrderNumber|Alphanumeric|Opcional|0..64|Número do pedido da loja.|
|SoftDescriptor|Alphanumeric|Opcional|0..13|Texto para ser exibido na fatura do portador, após o nome do estabelecimento comercial.|
|Cart|[Cart](#cart)|Sim|n/a|Informações sobre o carrinho de compras.|
|Shipping|[Shipping](#shipping)|Sim|n/a|Informações sobre a entrega do pedido."|
|Payment|[Payment](#payment)|Conditional|n/a|Informações sobre o pagamento do pedido.|
|Customer|[Customer](#customer)|Condicional|n/a|Informações sobre dados pessoais do comprador.|
|Options|[Options](#options)|Conditional|n/a|Informações sobre opções configuráveis do pedido.|

### Resposta

### Em caso de sucesso

```json
{
    "Settings": {
        "CheckoutUrl": "https://cieloecommerce.cielo.com.br/transacional/order/index?id=123",
        "Profile": "CheckoutCielo",
        "Version": 1
    }
}
```

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|Settings|[Settings](#settings)|Sim|n/a|Informações da resposta sobre a criação do pedido.|

### Em caso de erro

```json
{
    "message":"An error has occurred."
}
```

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|Message|String|Sim|1..254|Mensagem descritiva do erro|

### Erros de integração

Há dois tipos de erro que poderão ocorrer durante o processo de integração com o Checkout Cielo. São eles:

* **Antes da exibição da tela de Checkout** - Significa que houve algum dado erra do no envio da transação. Dados obrigatórios podem estar faltando ou no formato invalido. Aqui o lojista sempre vai receber um e-mail informando o que deu errado;
* **Depois da Exibição da tela de Checkout (quando a venda é finalizada)** - Significa que há algum impedimento de cadastro que limita a venda. Coisas como afiliação bloqueada, erro nos dados salvos no cadastro ou até problemas no próprio checkout.

## Botão

Integração via Botão é um método de compra usada sempre que não houver um “carrinho de compras” em sua loja ou quando se deseja associar uma compra rápida direta a um produto, como uma promoção numa homepage pulando a etapa do carrinho.

A integração via botão também pode ser usada para enviar um e-mail marketing, ou uma cobrança via e-mail, ou seja, adicionando o botão (HTML) referente ao produto/serviço a ser comprado/pago. Ou sempre que desejar disponibilizar uma venda rápida.

Para utilizar este recurso, é necessário cadastrar o produto que deseja vender, suas informações, e depois simplesmente copiar o código fonte gerado para este botão. A inclusão dos produtos é feita dentro do [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/), no menu de Produtos/Cadastrar Produto.

A inclusão dos produtos é feita dentro do [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/), no menu de Produtos/Cadastrar Produto.

![Integração com botão](/images/checkout-cielo-integracao-botao.png)

### Características do Botão

* Cada botão gerado serve somente para um determinado produto.
* O preço do produto não pode ser alterado na tela de Checkout
* Não é necessário o desenvolvimento de um carrinho
* O cadastro do produto é obrigatório para a criação do botão.

Cada botão possui um código único que só permite comprar aquele determinado produto nas condições de preço e frete cadastrado. Portanto, um fraudador não consegue alterar nenhuma destas informações na hora de submeter à compra, pois o Checkout Cielo vai buscar todos os dados do produto no cadastro do [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/), e valerão os dados do cadastro.

### Parâmetros para cadastro de produto

Abaixo seguem as informações necessárias para cadastrar um produto.

|Parâmetro|Descrição|Tamanho Min.|Tamanho Máx.|Obrigatório|
|---------|---------|------------|------------|-----------|
|Tipo do Produto|Indique se está vendendo um bem Material, um Serviço ou um bem Digital. Para bens Digitais, não será apresentada a opção de tipo de Frete.|n/a|n/a|Sim|
|SKU|Código de identificação do produto|1|50|Não|
|Título|Titulo do Produto|1|50|Sim|
|Descrição|Descrição do Produto|1|255|Sim|
|Preço|Valor total do pedido **em centavos** (ex.: R$1,00 =100).|11|14|Sim|
|Frete|Escolher dentre uma das opções de Frete (Correios, Frete Fixo, Frete Grátis, Retirar na loja, Sem Cobrança).|n/a|n/a|Sim|
|CEP de Origem|Esse campo só aparece para o frete tipo Correios, deve ser preenchido com o CEP de onde vai partir a mercadoria para fins de cálculo de frete.|9|9|Sim|
|Peso(kg)|Esse campo só aparece para o frete tipo Correios, deve ser preenchido com o peso do produto em kg para fins de cálculo de frete|n/a|n/a|Sim|
|Valor do Frete|Esse campo só aparece para o frete tipo Frete Fixo, e deve ser preenchido com o valor que o lojista especificar para seus produtos.|n/a|n/a|Sim|
|Método de envio|Esse campo só aparece para Tipo Produto igual a Material Físico e Tipo de Frete igual a Frete Fixo.|n/a|n/a|Sim|
|URL|Esse campo só aparece para Tipo Produto igual a Digital.|n/a|n/a|Sim|

#### Exemplo de Botão:

```html
<form method='post' action='https://cieloecommerce.cielo.com.br/transactional/Checkout/BuyNow' target='blank'>
    <input type='hidden' name='id' value=00000000-0000-0000-000000000000/><input type='image' name='submit' alt='Comprar' src='https://cieloecommerce.cielo.com.br /BackOffice/Content/images/botao_comprar_3.jpg' />
</form>
```

Adicionando o botão na sua página HTML você deve copiar o código HTML do botão criado e colocar no código HTML do seu site, conforme o exemplo abaixo.

<aside class="notice">O código deve ser inserido dentro da área adequada no seu HTML.</aside>

Cada botão possui um código único que só permite comprar aquele determinado produto nas condições de preço e frete cadastrado. Portanto, um fraudador não consegue alterar nenhuma destas informações na hora de submeter a compra, pois o Checkout Cielo vai buscar todos os dados do produto no cadastro do [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/), e valerão os dados do cadastro.

## Modo de teste do Checkout Cielo

O modo de teste Checkout Cielo é método de fazer testes de integração do Checkout Cielo com o seu site, sem o consumo de créditos. O modo de Teste Checkout Cielo permite realizar transações, testando a integração utilizando diferentes meios de pagamento simulados.

### Ativação do Modo de Teste.

O modo de teste pode ser ativado na aba Configurações:

![Modo de teste](/images/checkout-cielo-modo-teste.png)

Nessa área há um caixa de seleção, que quando marcada, habilitará o modo de teste do Checkout Cielo. O modo somente se iniciará quando a seleção for salva.

![Ativação Modo de teste](/images/checkout-cielo-modo-teste-ativacao.png)

Quando a opção for salva, uma tarja vermelha será exibida na parte superior da tela. Ela será exibida em todas as telas do [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/) e na tela de checkout do Checkout Cielo.

Essa tarja indica que a sua loja Checkout Cielo está agora operando em ambiente de teste, ou seja, toda a transação realizada nesse modo será considerada como teste.

![Modo de teste ativado](/images/checkout-cielo-modo-teste-ativado.png)

### Como transacionar no Modo de teste.

A realização de transações no modo de teste ocorre de forma normal. As informações da transação são enviadas via POST, utilizando os parâmetros como descrito no tópico [Integração com carrinho](#integração-carrinho-de-compras), entretanto, os meios de pagamentos a serem usados serão meios simulados.

Para realizar transações de teste com diferentes meios de pagamento, siga as seguintes regras:

**a - Transações com Cartão de crédito:**

Para testar cartões de crédito é necessário que dois dados importantes sejam definidos, o status da autorização do cartão e o retorno da analise de fraude.

**Status da Autorização do Cartão de Crédito**

|Digito final do Cartão|Status retornado|
|----------------------|----------------|
|0, 1, 2, 3 ou 4|Autorizado|
|5, 6, 7, 8 ou 9|Não autorizado|

* **Exemplo:** Transação autorizada, Alto Risco;
* **Numero do Cartão de credito:** 5404434242930107
* **Nome do Cliente:** Maria Alto

**b - Boleto Bancario**

Basta realizar o processo de compra normalmente sem nenhuma alteração no procedimento. O boleto gerado no modo de teste sempre será um boleto simulado.

**c - Debito online**

É necessário informa o status da transação de Debito online para que seja retornado o status desejado. Esse processo ocorre como no antifraude do cartão de crédito descrito acima, com a alteração do nome do comprador.

**Status do Débito**
|Sobre nome do cliente|Status|
|---------------------|------|
|Pago|Pago|
|Qualquer nome.|Não autorizado|

* **Exemplo:** Status não Autorizado.
* **Nome do Cliente:** Maria Pereira

**d - Transações de teste**

Todas as transações realizadas no modo de teste serão exibidas como transações normais na aba Pedidos do Checkout Cielo, entretanto, elas serão marcadas como transações de teste e não serão contabilizadas em conjunto com as transações realizadas fora do ambiente de teste.

![Transações de teste](/images/checkout-cielo-modo-teste-transacoes-de-teste.png)

Essas transações terão o simbolo de teste as diferenciando de suas outras transações. Elas podem ser capturadas ou canceladas utilizando os mesmos procedimentos das transações reais.

![Transações de teste](/images/checkout-cielo-modo-teste-transacoes-de-teste-cancelamento.png)

<aside class="notice">É muito importante que ao liberar sua loja para a realização de vendas para seus clientes que ela não esteja em modo de teste. Transações realizadas nesse ambiente poderão ser finalizadas normalmente, mas não serão descontadas do cartão do cliente e não poderão ser “transferidas” para o ambiente de venda padrão.</aside>

## Certificado Extended Validation

O Certificado Extended Validation para servidor web oferece autenticidade e integridade dos dados de um web site, proporcionando aos clientes das lojas virtuais a garantia de que estão realmente acessando o site que desejam, e não uma um site fraudador.

Empresas especializadas são responsáveis por fazer a validação do domínio e, dependendo do tipo de certificado, também da entidade detentora do domínio.

### O que é Certificado Extended Validation?

O Certificado EV foi lançado no mercado recentemente e garante um nível de segurança maior para os clientes das lojas virtuais.

Trata-se de um certificado de maior confiança e quando o https for acessado a barra de endereço ficará verde, dando mais confiabilidade aos visitantes do site.

Como instalar o Certificado Extended Validation no servidor da Loja?

Basta instalar os três arquivos a seguir na Trustedstore do servidor. A Cielo não oferece suporte para a instalação do Certificado. Caso não esteja seguro sobre como realizar a instalação do Certificado EV, então você deverá ser contatado o suporte do fornecedor do seu servidor.

* [Certificado Raiz](/attachment/Raiz.crt)
* [Certificado Intermediária](/attachment/Intermediaria.crt)
* [Certificado E-Commerce Cielo](/attachment/ecommerce.cielo.com.br.crt)

<aside class="notice">Caso seu servidor seja uma distribuição Linux e você tenha familiaridade e acesso ssh, então o <a href="/attachment/cielo.sh">Instalador Linux - cielo.sh</a> poderá ajudá-lo com a instalação. <strong>Apenas utilize o instalador se você souber o que está fazendo</strong>. Na dúvida, entre em contato com o suporte do fornecedor do seu servidor.</aside>

# URLs do Checkout Cielo

A loja deve configurar as três URLs (notificação, retorno e status) em seu [Backoffice Checkout Cielo](/Checkout-Backoffice/), na aba “Configurações”. Veja a tela abaixo:

![Configurações da loja](/images/checkout-configuracoes-loja.png)

* **URL de Retorno** – Página web na qual o comprador será redirecionado ao fim da compra. Nenhum dado é trocado ou enviado para essa URL. Essa URL apenas leva o comprador, após finalizar a compra, a uma página definida pela loja. Essa página deve ser configurada no [Backoffice Checkout Cielo](/Checkout-Backoffice/), aba “Configurações”
* **URL de Notificação** – Ao finalizar uma transação é enviado um POST HTTP com todos os dados da venda para a URL de Notificação, previamente cadastrada no [Backoffice Checkout Cielo](/Checkout-Backoffice/). **O POST de notificação é enviado apenas no momento que a transação é finalizada, independentemente se houve alteração do status da transação**. A URL deve conter uma página preparada para o receber os parâmetros  nos formatos definidos na tabela [Parâmetros para integração com POST de notificação](#parâmetros-para-integração-com-post-de-notificação) via a linguagem/modulo com o qual seu site foi desenvolvido, que receberá o POST HTTP.
* **URL de Mudança de Status** – Quando um pedido tiver seu status alterado, será enviando um post HTTP para a URL de Mudança de Status, previamente cadastrada no [Backoffice Checkout Cielo](/Checkout-Backoffice/). O POST de mudança de status não contem dados do carrinho, apenas dados de identificação do pedido. A URL deve conter uma pagina preparada para o receber os parâmetros  nos formatos definidos na tabela [Parâmetros para integração com o POST de Mudança de Status](#parâmetros-para-integração-com-post-de-mudança-de-status) via a linguagem/modulo com o qual seu site foi desenvolvido.

## URL de Retorno

A URL de Retorno é a que a Cielo utilizará para redirecionar o cliente de volta para a loja assim que o pagamento for concluído. Essa página da loja deverá estar preparada para receber o cliente ao fim do fluxo e avisá-lo que o processo foi concluído e que ele receberá mais informações em breve.

## URL de Notificação

 A URL de Notificação é a que a Cielo utilizará para enviar os dados da transação, do carrinho e da autorização ao fim do fluxo de integração. Ao receber a notificação, a loja terá todas as informações sobre o carrinho, pedido e poderá utilizar essas informações para alimentar seu sistema.

### Parâmetros para integração com POST de notificação

|Parâmetro|Descrição|Tipo do campo|Tamanho mínimo|Tamanho máximo|
|---------|---------|-------------|--------------|--------------|
|checkout_cielo_order_number|Identificador único gerado pelo CHECKOUT CIELO|Alfanumérico|1|32|
|amount|Preço unitário do produto, em centavos (ex: R$ 1,00 = 100)|Numérico|1|10|
|order_number|Número do pedido enviado pela loja|Alfanumérico|1|32|
|created_date|Data da criação do pedido (dd/MM/yyyy HH:mm:ss)|Alfanumérico|1|20|
|customer_name|Nome do consumidor. Se enviado, esse valor já vem preenchido na tela do CHECKOUT CIELO|Alfanumérico|1|289|
|customer_identity|Identificação do consumidor (CPF ou CNPJ) Se enviado, esse valor já vem preenchido na tela do CHECKOUT CIELO|Alfanumérico|1|14|
|customer_email|E-mail do consumidor. Se enviado, esse valor já vem preenchido na tela do CHECKOUT CIELO|Alfanumérico|1|64|
|customer_phone|Telefone do consumidor. Se enviado, esse valor já vem preenchido na tela do CHECKOUT CIELO|Numérico|1|11|
|discount_amount|Valor do desconto fornecido (enviado somente se houver desconto)|Numérico|1|10|
|shipping_type|Modalidade de frete|Numérico|1|1|
|shipping_name|Nome do frete|Alfanumérico|1|128|
|shipping_price|Valor do serviço de frete, em centavos (ex: R$ 10,00 = 1000)|Numérico|1|10|
|shipping_address_zipcode|CEP do endereço de entrega|Numérico|1|8|
|shipping_address_district|Bairro do endereço de entrega|Texto|1|64|
|shipping_address_city|Cidade do endereço de entrega|Alfanumérico|1|64|
|shipping_address_state|Estado de endereço de entrega|Alfanumérico|1|64|
|shipping_address_line1|Endereço de entrega|Alfanumérico|1|256|
|shipping_address_line2|Complemento do endereço de entrega|Alfanumérico|1|256|
|shipping_address_number|Número do endereço de entrega|Numérico|1|8|
|payment_method_type|Cód. do tipo de meio de pagamento|Numérico|1|1|
|payment_method_brand|Bandeira (somente para transações com meio de pagamento cartão de crédito)|Numérico|1|1|
|payment_method_bank|Banco emissor (Para transações de Boleto e Débito Automático)|Numérico|1|1|
|payment_maskedcredicard|Cartão Mascarado (Somente para transações com meio de pagamento cartão de crédito)|Alfanumérico|1|20|
|payment_installments|Número de parcelas|Numérico|1|1|
|payment_antifrauderesult|Status das transações de cartão de Crédito no Antifraude|Numérico|1|1|
|payment_boletonumber|Numero do boleto gerado|String|||
|payment_boletoexpirationdate|Data de vencimento para transações realizadas com boleto bancário|Numérico|1|10|
|payment_status|Status da transação|Numérico|1|1|
|tid|TID Cielo gerado no momento da autorização da transação|Alfanumérico|1|32|

<aside class="notice">A página destino do POST de Notificação deve seguir a formatação dos parâmetros com todos os nomes em MINUSCULO</aside>

### Recebimento do POST de notificação

* Quando acessada pelo servidor da Braspag, enviando o POST da tabela acima, a URL cadastrada para Notificação deverá exibir um código informando que recebeu a mudança de status e a processou com sucesso. **Código:**`<status>OK</status>`
* Se a URL for acessada pelo nosso servidor e não exibir o código de confirmação, o servidor irá tentar novamente por três vezes, a cada hora. Caso o `<status>OK</status>` ainda não seja exibido, será entendido que o servidor da loja não responde.
* A URL de Notificação somente pode utilizar **porta 80** (padrão para http) ou **porta 443** (padrão para https).

## URL de Mudança de Status

A URL de Mudança de Status é a que a Cielo utilizará para notificar a loja sobre as mudanças de status das transações. Uma mudança de status, de Autorizado para Cancelado, por exemplo, pode ocorrer a qualquer momento. Se o administrador da loja cancelar um pedido no Backoffice Cielo, então a Cielo enviará para a URL de Mudança de Status uma notificação semelhante a enviada para a URL de notificação. A única diferença dessa notificação é que não conterá os dados do carrinho, mas apenas do pedido e o novo status da autorização.

<aside class="warning">A URL de mudança de status é fornecida pelo lojista. Nessa URL serão postadas as informações de todos os pedidos que tiverem seu status alterado.</aside>

### Parâmetros para integração com POST de Mudança de Status

|Parâmetro|Descrição|Tipo do campo|Tamanho mínimo|Tamanho máximo|
|---------|---------|-------------|--------------|--------------|
|checkout_cielo_order_number|Identificador único gerado pelo CHECKOUT CIELO.|Alfanumérico|1|32|
|amount|Preço unitário do produto, em centavos (ex: R$ 1,00 = 100)|Numérico|1|10|
|order_number|Número do pedido enviado pela loja|Alfanumérico|1|32|
|payment_status|Status da transação|Numérico|1|1|

### Recebimento do POST de Mudança de Status

* Quando acessada pelo servidor da Braspag, enviando o POST, a URL cadastrada para Retorno de Mudança de Status, deverá exibir um código informando que recebeu a mudança de status e a processou com sucesso: `<status>OK</status>`
* Se a URL de mudança de status da loja for acessada pelo servidor da Braspag não exibir o código de confirmação, o servidor irá tentar novamente por três vezes.
* Caso o `<status>OK</status>` ainda não seja exibido, será entendido que o servidor da loja não responde, e será enviado um e-mail ao responsável pela loja, informando que o pedido em questão foi pago.
* Ou seja, o código fonte da página indicando Sucesso deverá conter APENAS `<status>OK</status>` **e nada mais**.

<aside class="notice">Na tela de pedidos, dentro de cada transação, há a opção de reenvio do POST de mudança de status. Basta clicar nos botões azuis, marcados na imagem abaixo</aside>

![Reenvio de notificação](/images/checkout-reenviar-notificacao.png)

# Parâmetros de integração

## Cart

```json
{
    "Discount": {},
    "Items": []
}
```

Parâmetro de requisição com informações sobre o carrinho de compras. Veja também o parâmetro [Item](#item)

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|Discount|[Discount](#discount)|Opcional|n/a|Informações do desconto sobre o carrinho de compras.|
|Items|[Item[]](#item)|Sim|n/a|Lista de itens do carrinho de compras *(deve conter no mínimo 1 item)*.|

### Discount

```json
{
    "Type": "Percent",
    "Value": 10
}
```

Parâmetro de requisição com informações sobre descontos.

<aside class="notice">Independentemente do tipo do desconto, ele deverá ser calculado antes da soma do valor do frete.</aside>

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|Type|Alphanumeric|Condicional|n/a|Tipo do desconto a ser aplicado: "Amount", "Percent". Obrigatório caso `Value` for maior ou igual a zero.|
|Value|Numeric|Condicional|0..18|Valor do desconto a ser aplicado *(pode ser valor absoluto ou percentual)*. Obrigatório caso `Type` for "Amount" ou "Percent".|

#### Discount.Type = Amount

Caso o tipo de desconto escolhido seja o `Amount`, deverá ser inserido o **valor em centavos**.
Ex.: 100 = 1,00.

#### Discount.Type Percent

Caso o tipo de desconto escolhido seja o `Percentual`, deverá ser inserido o **valor em número inteiro**.
Ex.: 10 = 10%.

### Item

```json
{
    "Name": "Nome do produto",
    "Description": "Descrição do produto",
    "UnitPrice": 100,
    "Quantity": 2,
    "Type": "Asset",
    "Sku": "Sku do item no carrinho",
    "Weight": 200
}
```

Parâmetro de requisição com informações sobre o item do carrinho de compras. Veja também o parâmetro [Cart](#cart)

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|Name|Alphanumeric|Sim|1..128|Nome do item no carrinho.|
|Description|Alphanumeric|Opcional|0.256|Descrição do item no carrinho.|
|UnitPrice|Numeric|Sim|1..18|Preço unitário do item no carrinho *(**em centavos.** Ex: R$ 1,00 = 100)*.|
|Quantity|Numeric|Sim|1..9|Quantidade do item no carrinho.|
|Type|Alphanumeric|Sim|n/a|Tipo do item no carrinho.|
|Sku|Alphanumeric|Opcional|0..32|Sku do item no carrinho.|
|Weight|Numeric|Condicional|0..9|Peso em gramas do item no carrinho.|

#### Tipos de item

|Tipo|Descrição|
|----|---------|
|Asset|Material Físico|
|Digital|Produtos Digitais|
|Service|Serviços|
|Payment|Outros pagamentos|

## Shipping

```json
{
    "Type": "Correios",
    "SourceZipCode": "14400000",
    "TargetZipCode": "11000000",
    "Address": {},
    "Services": []
}
```

Parâmetro de requisição com informações sobre endereço e serviço de entrega dos produtos.

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|Type|Alphanumeric|Sim|n/a|Tipo do frete: "Correios", "FixedAmount", "Free", "WithoutShippingPickUp", "WithoutShipping".|
|SourceZipCode|Numeric|Condicional|8|CEP de origem do carrinho de compras.|
|TargetZipCode|Numeric|Opcional|8|CEP do endereço de entrega do comprador.|
|Address|[Address](#address)|Opcional|n/a|Informações sobre o endereço de entrega do comprador.|
|Services|[Service[]](#service)|Condicional|n/a|Lista de serviços de frete.|

### Address

```json
{
    "Street": "Endereço de entrega",
    "Number": "123",
    "Complement": "",
    "District": "Bairro da entrega",
    "City": "Cidade de entrega",
    "State": "SP"
}
```

Parâmetro de requisição com informações sobre o endereço do comprador. Veja também o parâmetro [Customer](#customer)

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|Street|Alphanumeric|Sim|1..256|Rua, avenida, travessa, etc, do endereço de entrega do comprador.|
|Number|Alphanumeric|Sim|1..8|Número do endereço de entrega do comprador.|
|Complement|Alphanumeric|Opcional|0..256|Complemento do endereço de entrega do comprador.|
|District|Alphanumeric|Sim|1..64|Bairro do endereço de entrega do comprador.|
|City|Alphanumeric|Sim|1..64|Cidade do endereço de entrega do comprador.|
|State|Alphanumeric|Sim|2|Estado (UF) do endereço de entrega do comprador.|

### Service

```json
{
    "Name": "Serviço de frete",
    "Price": 123,
    "Deadline": 15
}
```

Parâmetro de requisição com informações sobre o serviço de frete que será utilizado.

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|Name|Alphanumeric|Sim|1..128|Nome do serviço de frete.|
|Price|Numeric|Sim|1..18|Preço do serviço de frete (em centavos. Ex: R$ 1,00 = 100).|
|Deadline|Numeric|Condicional|0..9|Prazo de entrega (em dias).|

## Payment

```json
{
    "BoletoDiscount": 0,
    "DebitDiscount": 10
}
```

Parâmetro de requisição com informações sobre o desconto para pagamento via boleto ou débito online.

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|BoletoDiscount|Numeric|Condicional|0..3|Desconto, em porcentagem, para pagamentos a serem realizados com boleto.|
|DebitDiscount|Numeric|Condicional|0..3|Desconto, em porcentagem, para pagamentos a serem realizados com débito online.|

## Customer

```json
{
    "Identity": 11111111111,
    "FullName": "Fulano Comprador da Silva",
    "Email": "fulano@email.com",
    "Phone": "11999999999"
}
```

Parâmetro de requisição com informações sobre o comprador. Veja também o parâmetro [Address](#address)

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|Identity|Numeric|Condicional|0..14|CPF ou CNPJ do comprador.|
|FullName|Alphanumeric|Condicional|0..288|Nome completo do comprador.|
|Email|Alphanumeric|Condicional|0..64|Email do comprador.|
|Phone|Numeric|Condicional|0..11|Telefone do comprador.|

## Options

```json
{
    "AntifraudEnabled": false
}
```

Parâmetro de requisição para configurar o sistema de anti-fraude para a transação.

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|AntifraudEnabled|Boolean|Conditional|n/a|Habilitar ou não a análise de fraude para o pedido.|

## Settings

Parâmetro de resposta, recebido em caso de sucesso.

```json
{
    "CheckoutUrl": "https://cieloecommerce.cielo.com.br/transacional/order/index?id=123",
    "Profile": "CheckoutCielo",
    "Version": 1 
}
```

|Campo|Tipo|Obrigatório|Tamanho|Descrição|
|-----|----|-----------|-------|---------|
|CheckoutUrl|Alphanumeric|Sim|1..128|URL de checkout do pedido. **Formato:** `https://cieloecommerce.cielo.com.br/transacional/order/index?id={id}`|
|Profile|Alphanumeric|Sim|1..16|Perfil do lojista: fixo "CheckoutCielo".|
|Version|Alphanumeric|Sim|1|Versão do serviço de criação de pedido *(versão: 1)*.|

# Configurações de Pagamento

## Cartão de Crédito

O Checkout Cielo aceita as principais bandeiras de crédito do Brasil e do mundo. São elas: Visa, MasterCard, American Express (Amex), Elo, Diners, Discover, JCB e Aura.

### Recebendo uma Venda de Cartão de Crédito

A partir da criação de uma transação, ela pode assumir diversos status. As transições de status podem ser realizadas através da troca de mensagens entre a loja e a Cielo, ou de forma automática, por exemplo, quando o prazo para a captura de transação autorizada expirar.

Pedidos por meio de cartão de crédito serão incluídos no [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/) como **“AUTORIZADO”** ou **“NÃO AUTORIZADO”**, dependendo do resultado da autorização na Cielo. Caso haja algum problema no processamento deste pedido (consumidor fechou a tela, por exemplo), ele constará como **“NÃO FINALIZADO”**.

### Análise de Fraude

Pedidos **“AUTORIZADOS”** serão enviados online, ou seja, no ato da venda, para análise da ferramenta de antifraude, quando  este desenvolvimento estiver devidamente padronizado na integração. O resultado desta análise será traduzido no campo **“Indicação AF”** no Relatório de Pedido, para cada pedido.

Esta análise indicará um **“BAIXO RISCO”** ou “ALTO RISCO” para a venda em questão. Esta sugestão é o que deve guiar a decisão de se confirmar  ou cancelar a venda. A analise será apresentada no “Detalhes do Pedido”, como abaixo:

![Análise de risco](/images/checkout-cielo-analise-risco.png)

## Cartão de débito

O Checkout Cielo aceita as principais bandeiras de cartão de débito do mercado: Visa e MasterCard. As transações de cartão de débito possuem como participantes os bancos emissores, que por sua vez usam dos mesmos recursos para transações online (token, cartão de senhas e etc) para o processo de autenticação. Consulte a relação de emissores participantes no Suporte Cielo e-Commerce.

* **E-mail**: [cieloecommerce@cielo.com.br](mailto:cieloecommerce@cielo.com.br)
* **Telefones**:
    * **Capitais**: (sem DDD) 4002.9700
    * **Demais Localidades**: 0800.570.1700
    * **Do exterior**: +55 (11) 2860.1348

A autenticação da transação garantirá uma segurança extra ao lojista contra contestações dos consumidores (chargeback). O produto débito obrigatoriamente exige uma transação autenticada, caso contrário, a transação não é autorizada. A autenticação é obrigatória para transações de débito e opcional para o crédito.

### Passo a passo da transação de cartão de débito:

1. Cliente acessa o internet banking
2. Digita a senha do cartão
3. Banco confirma a senha
4. Transação realizada

## Boleto

Todo boleto gerado (emitido) aparece com o status de “PENDENTE” no Relatório de Pedidos. Sua troca de status vai depender de ações manuais do proprio lojista. Para isso, acesse o [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/) (incluir link do manual) na seção Pedidos

### Possiveis Status do Boleto

* **PENDENTE** – boleto emitido pelo processo de transação. Status continua até alteração manual pelo lojista.
* **PAGO** – Status usado quando o botão "Conciliar” é ativado pelo lojista. Esse status pode ser revertido para pendente utiliando o Botão “Desfazer conciliação”.
* **EXPIRADO** – Status ativo após 10 dias da criação do boleto, caso esse não tenha sito conciliado nesse periodo. Boletos com status “EXPIRADO” podem ser conciliados.

### Conciliando um Boleto

Cabe ao lojista através de uma Conciliação Manual com seu extrato bancário, confirmar o pagamento do mesmo.

![Conciliando um boleto](/images/checkout-cielo-conciliar-boleto.png)

Para realizar a Conciliação você deve:

1. Acessar o relatório de pedidos no [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/);
2. Filtrar os pedidos por Meio de Pagamento “Boleto” e status “PENDENTE” e identificar o boleto em questão pelo Valor;
3. Clicar no sinal de + no final da linha para acessar a página de “Detalhes”;
4. Clicar no botão de “ Confirmar Pagamento ” e informar a data de pagamento, para seu futuro controle;

O pedido passa para status **PAGO**.

O Comprador também verá o pedido como **PAGO** no “Backoffice do Comprador”

Desfazendo a conciliação (pagamento) de um Boleto. Caso a conciliação tenha sido feito errada, basta:

1. Encontrar o Pedido;
2. Entrar no seu detalhe e clicar no botão “Desfazer Pagamento”;
3. O Pedido voltará para o Status de “PENDENTE”.

### Boletos Expirados

Se o boleto não for conciliado dentro de um prazo de 10 dias após seu vencimento, seu Status será alterado para **“EXPIRADO”**, para um melhor controle dos boletos vencidos. Boletos EXPIRADOS podem ser conciliados.

<aside class="notice">Validade do Boleto – Caso o boleto expire em um dia não útil, como Sábado, ele será valido até o próximo dia útil.</aside>

![Boleto](/images/checkout-cielo-boleto.png)

## Débito Online

Pedidos vendidos por meio de Débito online serão incluídos no [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/) como PENDENTE, PAGO, NÃO AUTORIZADO ou NÃO FINALIZADO, dependendo do resultado da autorização junto ao Banco.

* **Pendente** - Corresponde quando o comprador ao finalizar o pedido e não obtem resposta por parte do Banco, ou seja, não conseguir nem carregar a página do Banco para inserir os dados para o Débito.
* **Pago** - Corresponde quando o comprador conseguir realizar o pagamento do débito com sucesso.
* **Não Autorizado** - Apresentado para o Lojista quando o comprador tentar realizar uma transação via débito e não ter saldo para a transação.
* **Não Finalizado** - Apresentado para o Lojista caso o comprador tenha algum problema para finalizar o pagamento do meio Débito, seja fechando a janela do banco ou simplesmente nem chegando à tela do banco.

## Diferença entre estorno e cancelamento

* **Cancelamento:** é feito no mesmo dia da captura, devolvendo o limite ao cartão do comprador em até 72h conforme regras do banco emissor do cartão. Não é apresentado na fatura do comprador;
* **Estorno:** a partir do dia seguinte da captura, o valor é “devolvido” na fatura do comprador em até 300 dias. É apresentado na fatura do comprador.

## Captura/Cancelamento Automático

<aside class="notice">Veja a diferença entre cancelamento e estorno em <a href="#diferença-entre-estorno-e-cancelamento">Diferença entre estorno e cancelamento</a></aside>

### Captura automática

As vendas **“AUTORIZADAS”**, e com **“BAIXO RISCO”** na ferramenta de  antifraude poderão ser **CAPTURADAS** automaticamente pelo sistema. Para isso é preciso configurar no o [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/). Após essa configuração, o status apresentado será **“PAGO”**. Esta venda será então confirmada (capturada) na Cielo.

![Cancelamento e captura automático](/images/checkout-cielo-cancelamento-captura-automatico.png)

### Cancelamento Automático

As vendas “AUTORIZADAS”, e com “ALTO RISCO”  na ferramenta de  antifraude poderão ser CANCELADAS automaticamente pelo sistema. Para isso é preciso configurar no o [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/). Após essa configuração, o status apresentado será “CANCELADO”. Esta venda será então cancelada (desfeita) na Cielo.

![Cancelamento e captura automático](/images/checkout-cielo-cancelamento-captura-automatico.png)

<aside class="warning">Atenção! Você tem a opção de escolher a melhor integração para o seu negócio, a captura/cancelamento manual ou automático é feito diretamente pelo seu Backoffice.</aside>

![Cancelamento e captura automáticos](/images/checkout-cielo-anti-fraude-cancelamento-captura.png)

## Captura/Cancelamento manual

<aside class="notice">Veja a diferença entre cancelamento e estorno em <a href="#diferença-entre-estorno-e-cancelamento">Diferença entre estorno e cancelamento</a></aside>

As vendas **“AUTORIZADAS”** aguardam uma decisão de confirmação ou cancelamento. E esta decisão deve vir em conformidade com a análise de fraude, caso esta funcionalidade esteja devidamente parametrizada na integração.

A confirmação da venda deve ser feita pelo botão **CAPTURAR**, na aba  **“Pedidos”**, no [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/). Após a confirmação, o status mudará para **“PAGO”**. Esta venda será então confirmada (capturada) na Cielo.

Já o  cancelamento deve ser feito pelo botão **CANCELAR** na mesma seção No [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/). Após o cancelamento, o status mudará para **“CANCELADO”**. Esta venda será então cancelada (desfeita) na Cielo.

<aside class="warning">Atenção! Você tem até 5 dias pra confirmar a venda! Caso isso não seja feito ela não será mais válida na Cielo, e o limite reservado para sua loja/venda será liberado. Este é um procedimento padrão para todas as lojas.</aside>

<aside class="warning">Quando o prazo de confirmação da venda autorizada expira, os pedidos passarão automaticamente para o status “EXPIRADO”. Isso acontecerá no sexto dia após a data de autorização (data da venda)</aside>

## Estorno de Venda

Caso a venda já tenha sido confirmada (status PAGO) ela pode ser ainda, futuramente, estornada. Para isso, basta clicar no botão no Detalhe do Pedido.

### Vendas de Cartões de Crédito Expiradas

Quando o prazo de confirmação da venda autorizada expira, os pedidos passarão automaticamente para o status “EXPIRADO”. Isso acontecerá no sexto dia após a data de autorização (data da venda)

## Chargeback

O consumidor (comprador) pode por algum motivo cancelar a compra diretamente com o banco emissor do cartão de crédito. Caso isso ocorra  o lojista receberá da Cielo um aviso de Chargeback de “Não Reconhecimento de compra” ou caso tenha havido uma compra com cartão fraudado, você recebera um aviso de Chargeback por “Fraude”.

![Chargeback](/images/checkout-cielo-chargeback.png)

Essa comunicação não é feita via o [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/), mas sim pelo extrato de venda da Cielo, destacada como um ajuste financeiro. O extrato de vendas está disponivel no site da Cielo [www.cielo.com.br na aba “Acessar Minha conta”](https://www.cielo.com.br/minha-conta).

![Acessar minha conta](/images/acessar-minha-conta.png)

Após esse recebimento, no próprio site da Cielo  é possivel acessar o [Backoffice Cielo Checkout](http://developercielo.github.io/Checkout-Backoffice/) e sinalizar o pedido como tendo recebido um Chargeback, pra seu melhor controle. Basta entrar no Detalhe do Pedido e clicar no botão “ChargeBack”, e seu status passará a ser “CHARGEBACK”.

## Frete

O Checkout Cielo suporta diferentes tipos de frete, que podem ser utilizados de maneira diferenciada de acordo com as opções oferecidas em sua loja. As opções disponíveis são:

* Correios
* Frete Fixo
* Frete Grátis
* Sem Frete

A maneira e o tipo de frete que ficará ativo em sua loja é configurado no Backoffice do Checkout Cielo. Devido ao aspecto mais técnico, sugerimos que as configurações de frete sejam feitas pelo desenvolvedor. Diferentes métodos de cálculo de frete:

### Cálculo de frete próprio

É possível selecionar 1 ou mais opções de frete. Elas serão apresentadas ao consumidor de acordo com a sua escolha entre as opções disponíveis. O valor selecionado pelo consumidor será adicionado ao valor total da compra.

### Contrato próprio com os Correios

* O Checkout Cielo usará este número de contrato para fazer o cálculo de frete, utilizando assim a tabela de frete que você possui acordada junto aos Correios. Desse modo, o checkout apresentará todas as opções de frete dos Correios (Sedex, Sedex 10, Sedex. Hoje e PAC, entre outras) para o consumidor escolher, de acordo com o CEP de destino digitado. O valor selecionado pelo Consumidor será adicionado ao valor total da compra.
* Usar o cálculo de frete dos Correios do Checkout Cielo. O Checkout Cielo apresentará todas as opções de frete dos Correios (Sedex, Sedex 10, Sedex Hoje e PAC, entre outras) para o consumidor escolher, de acordo com o CEP de destino digitado. O valor selecionado pelo consumidor será adicionado ao valor total da compra.
* Seleção do Frete no carrinho e não no Checkout Cielo. O Checkout Cielo apresentará somente a tela de escolha do meio de pagamento para o consumidor. O valor do frete já estará embutido no valor final

<aside class="notice">O consumidor não poderá alterar o endereço de entrega na tela do Checkout Cielo</aside>