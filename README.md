# Safe2Pay .NET SDK

![Safe2Pay](https://safe2pay.com.br/static/img/banner-github.png)

##### Biblioteca de integração em .NET para o [Safe2Pay](https://safe2pay.com.br/).

Recomendamos a utilização do [pacote da galeria do NuGet](https://www.nuget.org/packages/Safe2Pay/), para manter-se com a versão mais atualizada da biblioteca, com todas as funcionalidades atuais e as que estão por vir!

## Instalação

[![NuGet version](https://img.shields.io/nuget/vpre/Safe2Pay.svg?style=flat-square)](https://www.nuget.org/packages/Safe2Pay/)

Pelo Visual Studio, buscando por 'Safe2Pay'.

Usando o .NET CLI: `dotnet add package Safe2Pay`.

Usando o Package Manager: `Install-Package Safe2Pay`.

## Principais recursos

* Conta-Corrente (`AccountRequest`)
    * Consulta e atualização de dados bancários
    * Detalhamento e calendário de recebimentos
* Tokenização de cartão de crédito (`TokenRequest`)
* **Geração** de Transações (`CheckoutRequest`)
    * Boleto bancário
    * Cartão de crédito e de débito
    * Criptomoedas
    * Débito em conta
    * Geração de carnês e carnês em lote
    * Transferência bancária
* **Operações** com Transações (`TransactionRequest`)
    * Consulta de transações
    * Listagem de transações
    * Estorno de cartão de crédito e cartão de débito
    * Liberação e cancelamento de boleto bancário
    * Consulta e cancelamento de carnês
* Débito em conta (`DebitAccountRequest`)
    * Consulta e cancelamento de débitos
* Solicitações de cobrança (`InvoiceRequest`)
    * Geração, cancelamento e substituição
    * Consulta e listagem
    * Reenvio
* Gestão de Marketplace (`MarketplaceRequest`)
    * Inclusão, atualização e exclusão de subcontas
    * Consulta e listagem de subcontas
* Gestão de Planos (`PlanRequest`) e Adesões (`SubscriptionRequest`)
    * Inclusão e atualização de planos e adesões
    * Consultas e listagens
* Transferência bancária (`TransferRequest`)
    * Consulta e listagens


## Requisitos

.NET Standard 1.1+ ou
.NET Framework 4.5+ ou
.NET Core 1.0+

## Utilização

A integração com a API do Safe2Pay se dá pelo modelo RESTful, de forma a realizar a transferência segura e simplificada dos dados pelo formato JSON. Para facilitar o envio dos dados, deve-se montar um objeto para envio baseado nos modelos disponíveis, com exemplos abaixo, e a própria chamada do método desejado realizará o tratamento e conversão deste objeto para JSON. 

### Configuração

Antes de iniciar a utilização da biblioteca, é necessário informar os dados básicos de autenticação na API, **não esquecendo de utilizar o Token e da Secret Key correspondentes ao ambiente definido, se Produção ou Sandbox**. Esta configuração está dentro da classe `Config` e complementará a chamada da API com os dados da sua empresa e deve ser utilizada na inicialização da classe com os métodos desejados. Segue exemplo abaixo:

```
var config = new Config(
    token: "PREENCHA_COM_SEU_TOKEN",
    secret: "PREENCHA_COM_SUA_SECRET_KEY");

var checkout = new Checkout(config);
//Utilização dos métodos da classe Checkout...
```

### Tratamento das respostas da API

Após o envio, a própria chamada devolverá a resposta em um objeto completo com as propriedades desta, onde um cast das classes de resposta permitirá o tratamento das propriedades do objeto de retorno de forma simplificada, sem a necessidade de criar os mesmos modelos em seu projeto. 

* `AccountResponse` para operações com os métodos de conta-corrente em `AccountRequest`;
* `CheckoutResponse` para operações com transações em `CheckoutRequest`, `TransactionRequest`, `DebitAccountRequest`, `TransferRequest` ou `TokenRequest`;
* `InvoiceResponse` para operações com solicitações de cobrança em `InvoiceRequest`;
* `MarketplaceResponse` para operações de gestão de subcontas/marketplaces em `MarketplaceRequest`;
* `PlanResponse` e `SubscriptionResponse` para operações com planos `PlanRequest`  e/ou adesões `SubscriptionRequest`;

Exemplo:

```
var checkout = new CheckoutRequest(config);
var response = (CheckoutResponse)checkout.Credit(transaction); 

Console.WriteLine($"Transação {response.IdTransaction} gerada com sucesso!");
```

## Pagamentos / Transações

O objeto esperado para uma transação deve seguir o modelo abaixo:

```
var transaction = new Transaction<object>
{
	IsSandbox = true, //Definir com base no Token utilizado
		
	PaymentMethod = new PaymentMethod { Code = "CODIGO_DA_FORMA_DE_PAGAMENTO" },
	PaymentObject = { /*CORPO DO OBJETO ESPERADO PARA A FORMA DE PAGAMENTO*/ };
	Application = "NOME_DA_SUA_APLICAÇÃO",
	Vendor = "VENDEDOR",
	Reference = "REFERÊNCIA",
	CallbackUrl = "https://callbacks.exemplo.com/api/notify",
	Customer = new Customer
	{
		Name = "Destinatário da Transação",
		Identity = "99999999999999", //CPF ou CNPJ do Destinatário
		Email = "email@empresa.com.br",
		Address = new Address
		{
			Street = "Endereço do Destinatário",
			Number = "Número 123",
			District = "Bairro",
			ZipCode = "99999999", 
			CityName = "Porto Alegre",
			StateInitials = "RS",
			CountryName = "Brasil"
		}
	},
	Products = new List<Product>
	{
		new Product {Code = "001", Description = "Produto 1", UnitPrice = 10M, Quantity = 1M},
		new Product {Code = "002", Description = "Produto 2", UnitPrice = 9.99M, Quantity = 2M},
	}
};
```

A informação da forma de pagamento é dada por meio da propriedade `PaymentMethod`, onde deve ser informado o código correspondente ao método desejado:

```
1 = Boleto bancário
2 = Cartão de crédito
3 = Criptomoedas
4 = Cartão de débito
10 = Débito em conta
```

O retorno do envio da transação trará um status para esta. [Consulte todos os status disponíveis nesta lista](https://developers.safe2pay.com.br/reference/transaction_info).

#### Boleto Bancário

```
var bankSlip = new Transaction<BankSlip>
{
	PaymentMethod = new PaymentMethod { Code = "1" },
	PaymentObject = new BankSlip
	{
		//OBRIGATÓRIO 
		DueDate = new DateTime(2019, 07, 31), //Data de vencimento do Boleto Bancário
		
		//OPCIONAIS
		Message = new List<string> { "Mensagem 1", "Mensagem 2" }, //Mensagens (máx. 10) que serão impressas no Boleto
		Instruction = "Instrução", //Mensagem na área de "Instruções"
		CancelAfterDue = true, //Se o boleto deve ser cancelado após a data de vencimento
		IsEnablePartialPayment = false, //Se o boleto aceita pagamento diferente do valor de registro
		InterestRate = 1m, //Valor da Taxa de Juros
		PenaltyRate = 1m, //Valor da Multa
		DaysBeforeCancel = 30 //Prazo para baixa do boleto após o vencimento
	},
	//DEMAIS DADOS DO OBJETO DA TRANSAÇÃO, CONFORME O MODELO ACIMA
};
```

O método `BankSlip` é o responsável pelo envio de uma transação com boleto e está na classe `Checkout`.

```
var bankSlip = new Transaction<BankSlip> { /*CORPO DO OBJETO DA TRANSAÇÃO*/ };

var checkout = new CheckoutRequest(config);
var response = (CheckoutResponse)checkout.BankSlip(bankSlip);

Console.WriteLine($"Transação {response.IdTransaction} gerada com sucesso.");

Console.WriteLine($"Pagamento pendente. Copie a linha digitável para realizar o pagamento do boleto: {response.DigitableLine}");
//OU...
Console.WriteLine($"Pagamento pendente. O link para download e impressão do boleto é: {response.BankSlipUrl}");
```

#### Cartão de Crédito

Para realizar a tokenização dos dados do cartão de crédito de um cliente, deve-se usar o método `Tokenize`, em `TokenRequest`, que retorna uma string com o token gerado para posterior utilização segura em uma transação.

```
var card = new CreditCard
{
	CardNumber = "4111111111111111",
	Holder = "Titular do Cartão",
	ExpirationDate = "12/2021",
	SecurityCode = "999"
};

var checkout = new TokenRequest(config);
var response = (CheckoutResponse)checkout.Tokenize(card);

Console.WriteLine($"Token '{teste.Token}' criado com sucesso!");
```

Formato do objeto de envio de uma transação por cartão de crédito:

```
var credit = new Transaction<CreditCard>
{
	PaymentMethod = new PaymentMethod { Code = "2" },
	PaymentObject = new CreditCard
	{
		//OPCIONAL - Número de vezes que a venda será parcelada
		InstallmentQuantity = 3,

		//Caso os dados do cartão já estejam tokenizados, informar apenas o token
		Token = "INFORMAR_O_CARTÃO_TOKENIZADO"
				
		//Ou os dados completos do cartão de crédito do cliente
		CardNumber = "4111111111111111",
		Holder = "Titular do Cartão",
		ExpirationDate = "12/2021",
		SecurityCode = "999"
	},
	//DEMAIS DADOS DO OBJETO DA TRANSAÇÃO, CONFORME O MODELO ACIMA
};
```

O método `Credit` é o responsável pelo envio de uma transação com cartão de crédito e está na classe `Checkout`:

```
var credit = new Transaction<CreditCard> { /*CORPO DO OBJETO DA TRANSAÇÃO*/ };

var checkout = new CheckoutRequest(config);
var response = (CheckoutResponse)checkout.Credit(credit);

Console.WriteLine($"Transação {response.IdTransaction} gerada com sucesso!");

Console.WriteLine(response.Status.Equals("3")
	? $"Transação {response.IdTransaction} autorizada!"
	: $"Ocorreu um erro: {response.Message}"); // Se status != 3, exibir a mensagem com o erro ocorrido
```

Para realizar o estorno de uma transação realizada por cartão de crédito, deve-se utilizar o método `RefundCredit`, dentro da classe `TransactionRequest`.

```
var transaction = new TransactionRequest(config);
var refund = transaction.RefundCredit(response.IdTransaction); //Utilizando a transação anterior como exemplo

if (refund.isCancelled) Console.WriteLine("Estorno realizado com sucesso!");
```

#### Bitcoin

```
var transaction = new Transaction<Bitcoin>
{
	PaymentMethod = new PaymentMethod { Code = "3" },
	//DEMAIS DADOS DO OBJETO DA TRANSAÇÃO, CONFORME O MODELO ACIMA
};
```
Para uma transação por Bitcoin, basta informar o código do método de pagamento (`Code = "3"`). A propriedade `PaymentObject` **não é necessária**. 

O método `Bitcoin` é o responsável pelo envio de uma transação com Bitcoin e está na classe `Checkout`.

```
var bitcoin = new Transaction<Bitcoin> { /*CORPO DO OBJETO DA TRANSAÇÃO*/ };

var checkout = new CheckoutRequest(config);
var response = (CheckoutResponse)checkout.Bitcoin(bitcoin);

Console.WriteLine($"Transação {response.IdTransaction} gerada com sucesso.");

Console.WriteLine($"Pagamento pendente. Por favor, escaneie o código da imagem {response.QrCode} para realizar o pagamento!");
//OU...
Console.WriteLine($"Pagamento pendente. Por favor, realizar o envio de {response.AmountBTC} BTC para o endereço {response.WalletAddress} para completar a transação!");
```

#### Cartão de Débito

```
var debit = new Transaction<DebitCard>
{
	PaymentMethod = new PaymentMethod { Code = "4" },
	PaymentObject = new DebitCard
	{
		//Deve ser verdadeiro para a transação ser finalizada no internet banking da Instituição Bancária do cliente
		Authenticate = true,

		//Dados completos do cartão de débito do cliente
		CardNumber = "4111111111111111",
		Holder = "Titular do Cartão",
		ExpirationDate = "12/2021",
		SecurityCode = "999"
	},
	//DEMAIS DADOS DO OBJETO DA TRANSAÇÃO, CONFORME O MODELO ACIMA
};
```

O método `Debit` é o responsável pelo envio de uma transação com cartão de débito e está na classe `Checkout`:

```
var debit = new Transaction<DebitCard> { /*CORPO DO OBJETO DA TRANSAÇÃO*/ };

var checkout = new CheckoutRequest(config);
var response = (CheckoutResponse)checkout.Debit(debit);

Console.WriteLine($"Transação {response.IdTransaction} gerada com sucesso!");

Console.WriteLine($"Pagamento pendente. Por favor, acesse a página {response.AuthenticationUrl} para finalizar o pagamento através do Internet Banking de sua Instituição Bancária!");
```

Para realizar o estorno de uma transação realizada por cartão de crédito, deve-se utilizar o método `RefundDebit`, dentro da classe `TransactionRequest`.

```
var transaction = new TransactionRequest(config);
var refund = transaction.RefundDebit(response.IdTransaction); //Utilizando a transação anterior como exemplo

if (refund.isCancelled) Console.WriteLine("Estorno realizado com sucesso!");
```

## Solicitações de Cobrança / Vendas Rápidas

Na classe `InvoiceRequest` estão os métodos disponíveis pela geração e tratamento de solicitações de cobrança. **Solicitações de cobrança não podem ser geradas em Sandbox.**

O objeto para uma nova venda deve ser montado seguindo o modelo abaixo:

```
var singleSale = new SingleSale
{
    Customer = new Customer
    {
        Name = "Destinatário da Cobrança",
        Identity = "01579286000174", //CPF ou CNPJ do Destinatário
        Email = "email@empresa.com.br",
        Address = new Address
        {
            Street = "Endereço do Destinatário",
            Number = "Número 123",
            District = "Bairro",
            ZipCode = "99999999",
            CityName = "Porto Alegre",
            StateInitials = "RS",
            CountryName = "Brasil"
        }
    },
    Products = new List<Product>
    {
        new Product {Code = "001", Description = "Produto 1", UnitPrice = 1.99M, Quantity = 1M},
        new Product {Code = "002", Description = "Produto 2", UnitPrice = 2.99M, Quantity = 2M},
        new Product {Code = "003", Description = "Produto 3", UnitPrice = 3.99M, Quantity = 3M}
    },
    ExpirationDate = new DateTime(2019, 07, 10), //Data de expiração da cobrança
    DiscountAmount = 10, //Valor do desconto, em Reais, caso exista
    Instruction = "Solicitação de cobrança pelos produtos 1, 2 e 3.",
    PaymentMethods = new List<PaymentMethod>
    {
        new PaymentMethod { CodePaymentMethod = "1" }, //Boleto
        new PaymentMethod { CodePaymentMethod = "2" }, //Crédito
        new PaymentMethod { CodePaymentMethod = "3" }, //Bitcoin
        new PaymentMethod { CodePaymentMethod = "4" }, //Débito
    },
    DueDate = new DateTime(2019, 07, 10), //Vencimento do Boleto Bancário, caso  habilitado
    InterestAmount = 2, //Valor dos Juros do Boleto Bancário, caso  habilitado
    PenaltyAmount = 3, //Valor da Multa do Boleto Bancário, caso  habilitado
    Messages = new List<string> { "Mensagem 1", "Mensagem 2" },
    Emails = new List<string> { "email1@empresa.com.br", "email2@company.com" } //E-mails para envio da cobrança
};

var invoice = new InvoiceRequest(config);
var response = (InvoiceResponse)invoice.New(singleSale);

Console.WriteLine($"Solicitação {response.SingleSaleHash} gerada com sucesso!");
Console.WriteLine($"Siga o link para realizar o pagamento: {response.SingleSaleUrl} ");
```

Para realizar o cancelamento de uma solicitação de cobrança, basta realizar uma chamada para o método `Cancel`, informando o hash gerado para a solicitação desejada. A resposta será um booleano com a confirmação do cancelamento.

```
//Utilizando a resposta do método anterior como variável
var saleToCancel = new SingleSale { SingleSaleHash = response.SingleSaleHash }; 

var invoice = new InvoiceRequest(config);
var confirmation = invoice.Cancel(saleToCancel);

if (confirmation) Console.WriteLine("Cobrança cancelada com sucesso!");
```

## Histórico de versões / Informações adicionais / Contato

**Changelog**: [clique aqui](https://github.com/SafeToPay/.NET/blob/master/CHANGELOG.md).

A orientação sobre a utilização da API também está disponível na documentação de referência da API, [disponível aqui]([https://developers.safe2pay.com.br/](https://developers.safe2pay.com.br/)), porém salientamos que ela se encontra em atualização para a nova versão da API e, por isso, recomendamos a utilização do [pacote da galeria do NuGet](https://www.nuget.org/packages/Safe2Pay), para que você esteja sempre com a versão mais atualizada!

Em caso de dúvidas, [ficamos à disposição em nossos canais](https://safe2pay.com.br/contato) ou diretamente pelo e-mail <integracao@safe2pay.com.br>.
