# Safe2Pay .NET SDK

![Safe2Pay](https://safe2pay.com.br/static/img/banner-github.png)

##### Biblioteca de integra��o em .NET para o [Safe2Pay](https://safe2pay.com.br/Safe2Pay/).

Recomendamos a utiliza��o do pacote da galeria do NuGet, para manter-se com a vers�o mais atualizada da biblioteca, com todas as funcionalidades atuais e as que est�o por vir!

## Instala��o

[![NuGet version](https://img.shields.io/nuget/vpre/Safe2Pay.svg?style=flat-square)](https://www.nuget.org/packages/Safe2Pay)

Pelo Visual Studio, buscando por 'Safe2Pay'. 
Usando o .NET CLI: `dotnet add package Safe2Pay`.
Usando o Package Manager: `Install-Package Safe2Pay`.

## Requisitos

.NET Standard 1.1+ ou
.NET Framework 4.5+ ou
.NET Core 1.0+

## Utiliza��o

A integra��o com a API do Safe2Pay se d� pelo modelo RESTful, de forma a realizar a transfer�ncia segura e simplificada dos dados pelo formato JSON. Para facilitar o envio dos dados, deve-se montar um objeto para envio baseado nos modelos dispon�veis, com exemplos abaixo, e a pr�pria chamada do m�todo desejado realizar� o tratamento e convers�o deste objeto para JSON. 

### Configura��o

Antes de iniciar a utiliza��o da biblioteca, � necess�rio informar os dados b�sicos de autentica��o na API e tamb�m em qual ambiente ser� feita sua utiliza��o, se em Produ��o (`isSandbox: false`) ou em Sandbox (`isSandbox: true`), **n�o esquecendo de utilizar os dados do Token e da Secret Key correspondentes ao ambiente definido**. Esta configura��o est� dentro da classe `Config` e complementar� a chamada da API com os dados da sua empresa e deve ser utilizada na inicializa��o da classe com os m�todos desejados. Segue exemplo abaixo:

```
var config = new Config(
    token: "PREENCHA_COM_SEU_TOKEN",
    secret: "PREENCHA_COM_SUA_SECRET_KEY",
    isSandbox: true );

var checkout = new Checkout(config);
//Utiliza��o dos m�todos em Checkout...
```

### Tratamento das respostas da API

Ap�s o envio, a pr�pria chamada devolver� a resposta em um objeto completo, onde, com o cast simples das classes `CheckoutResponse` (para transa��es) ou `InvoiceResponse` (para solicita��es de cobran�a) permitir� o tratamento das propriedades do objeto de retorno de forma simplificada, sem a necessidade de criar os mesmos modelos em seu projeto.

```
var checkout = new Checkout(config);
var response = (CheckoutResponse)checkout.Credit(transaction); 

Console.WriteLine($"Transa��o {response.IdTransaction} gerada com sucesso!");
```

## Pagamentos / Transa��es

Existem duas classes respons�veis pela gera��o e tratamentos de transa��es, `Checkout` e `Transaction`, onde cada uma possui os m�todos dispon�veis para cada .

O objeto esperado para uma transa��o deve seguir o modelo abaixo:

```
var transaction = new Transaction<object>
{
	PaymentMethod = new PaymentMethod { Code = "CODIGO_DA_FORMA_DE_PAGAMENTO" },
	PaymentObject = { /*CORPO DO OBJETO ESPERADO PARA A FORMA DE PAGAMENTO*/ };
	Application = "NOME_DA_SUA_APLICA��O",
	Vendor = "VENDEDOR",
	Reference = "REFER�NCIA",
	CallbackUrl = "https://callbacks.exemplo.com/api/notify",
	Customer = new Customer
	{
		Name = "Destinat�rio da Transa��o",
		Identity = "99999999999999", //CPF ou CNPJ do Destinat�rio
		Email = "email@empresa.com.br",
		Address = new Address
		{
			Street = "Endere�o do Destinat�rio",
			Number = "N�mero 123",
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

A informa��o da forma de pagamento � dada por meio da propriedade `PaymentMethod`, onde deve ser informado o c�digo correspondente ao m�todo desejado:

1. Boleto Banc�rio;
2. Cart�o de Cr�dito;
3. Bitcoin;
4. Cart�o de D�bito

O retorno do envio da transa��o trar� um status para esta, que pode ser igual a:

```
1 = PENDENTE
2 = PROCESSAMENTO
3 = AUTORIZADO
4 = DISPON�VEL
5 = EM DISPUTA
6 = DEVOLVIDO
7 = BAIXADO
8 = RECUSADO
11 = LIBERADO
12 = EM CANCELAMENTO
```

#### Boleto Banc�rio

```
var bankSlip = new Transaction<BankSlip>
{
	PaymentMethod = new PaymentMethod { Code = "1" },
	PaymentObject = new BankSlip
	{
		//OBRIGAT�RIO 
		DueDate = new DateTime(2019, 07, 31), //Data de vencimento do Boleto Banc�rio
		
		//OPCIONAIS
		Message = new List<string> { "Mensagem 1", "Mensagem 2" }, //Mensagens (m�x. 10) que ser�o impressas no Boleto
		Instruction = "Instru��o", //Mensagem na �rea de "Instru��es"
		CancelAfterDue = true, //Se o boleto deve ser cancelado ap�s a data de vencimento
		IsEnablePartialPayment = false, //Se o boleto aceita pagamento diferente do valor de registro
		InterestRate = 1m, //Valor da Taxa de Juros
		PenaltyRate = 1m, //Valor da Multa
		DaysBeforeCancel = 30 //Prazo para baixa do boleto ap�s o vencimento
	},
	//DEMAIS DADOS DO OBJETO DA TRANSA��O, CONFORME O MODELO ACIMA
};
```

O m�todo `BankSlip` � o respons�vel pelo envio de uma transa��o com boleto e est� na classe `Checkout`.

```
var bankSlip = new Transaction<BankSlip> { /*CORPO DO OBJETO DA TRANSA��O*/ };

var checkout = new Checkout(config);
var response = (CheckoutResponse)checkout.BankSlip(bankSlip);

Console.WriteLine($"Transa��o {response.IdTransaction} gerada com sucesso.");

Console.WriteLine($"Pagamento pendente. Copie a linha digit�vel para realizar o pagamento do boleto: {response.DigitableLine}");
//OU...
Console.WriteLine($"Pagamento pendente. O link para download e impress�o do boleto �: {response.BankSlipUrl}");
```

#### Cart�o de Cr�dito

Para realizar a tokeniza��o dos dados do cart�o de cr�dito de um cliente, deve-se usar o m�todo `Tokenize`, tamb�m dentro de Checkout, que retorna uma string com o Token gerado para posterior utiliza��o em uma transa��o.

```
var card = new CreditCard
{
	CardNumber = "4111111111111111",
	Holder = "Titular do Cart�o",
	ExpirationDate = "12/2021",
	SecurityCode = "999"
};

var checkout = new Checkout(config);
var response = (CheckoutResponse)checkout.Tokenize(card);

Console.WriteLine($"Token '{teste.Token}' criado com sucesso!");
```

Formato do objeto de envio de uma transa��o por cart�o de cr�dito:

```
var credit = new Transaction<CreditCard>
{
	PaymentMethod = new PaymentMethod { Code = "2" },
	PaymentObject = new CreditCard
	{
		//OPCIONAL - N�mero de vezes que a venda ser� parcelada
		InstallmentQuantity = 3,

		//Caso os dados do cart�o j� estejam tokenizados, informar apenas o token
		Token = "INFORMAR_O_CART�O_TOKENIZADO"
				
		//Ou os dados completos do cart�o de cr�dito do cliente
		CardNumber = "4111111111111111",
		Holder = "Titular do Cart�o",
		ExpirationDate = "12/2021",
		SecurityCode = "999"
	},
	//DEMAIS DADOS DO OBJETO DA TRANSA��O, CONFORME O MODELO ACIMA
};
```

O m�todo `Credit` � o respons�vel pelo envio de uma transa��o com cart�o de cr�dito e est� na classe `Checkout`:

```
var credit = new Transaction<CreditCard> { /*CORPO DO OBJETO DA TRANSA��O*/ };

var checkout = new Checkout(config);
var response = (CheckoutResponse)checkout.Credit(credit);

Console.WriteLine($"Transa��o {response.IdTransaction} gerada com sucesso!");

Console.WriteLine(response.Status.Equals("3")
	? $"Transa��o {response.IdTransaction} autorizada!"
	: $"Ocorreu um erro: {response.Message}"); // Se status != 3, exibir a mensagem com o erro ocorrido
```

Para realizar o estorno de uma transa��o realizada por cart�o de cr�dito, deve-se utilizar o m�todo `Refund`, dentro da classe `Transaction`.

```
var transaction = new Transaction(config);
var refund = transaction.Refund(response.IdTransaction); //Utilizando a transa��o anterior como exemplo

if (refund.isCancelled) Console.WriteLine("Estorno realizado com sucesso!");
```

#### Bitcoin

```
var transaction = new Transaction<Bitcoin>
{
	PaymentMethod = new PaymentMethod { Code = "3" },
	//DEMAIS DADOS DO OBJETO DA TRANSA��O, CONFORME O MODELO ACIMA
};
```
Para uma transa��o por Bitcoin, basta informar o c�digo do m�todo de pagamento (`Code = "3"`). A propriedade `PaymentObject` **n�o � necess�ria**. 

O m�todo `Bitcoin` � o respons�vel pelo envio de uma transa��o com Bitcoin e est� na classe `Checkout`.

```
var bitcoin = new Transaction<Bitcoin> { /*CORPO DO OBJETO DA TRANSA��O*/ };

var checkout = new Checkout(config);
var response = (CheckoutResponse)checkout.Bitcoin(bitcoin);

Console.WriteLine($"Transa��o {response.IdTransaction} gerada com sucesso.");

Console.WriteLine($"Pagamento pendente. Por favor, escaneie o c�digo da imagem {response.QrCode} para realizar o pagamento!");
//OU...
Console.WriteLine($"Pagamento pendente. Por favor, realizar o envio de {response.AmountBTC} BTC para o endere�o {response.WalletAddress} para completar a transa��o!");
```

#### Cart�o de D�bito

```
var debit = new Transaction<DebitCard>
{
	PaymentMethod = new PaymentMethod { Code = "4" },
	PaymentObject = new DebitCard
	{
		//Deve ser verdadeiro para a transa��o ser finalizada no internet banking da Institui��o Banc�ria do cliente
		Authenticate = true,

		//Dados completos do cart�o de d�bito do cliente
		CardNumber = "4111111111111111",
		Holder = "Titular do Cart�o",
		ExpirationDate = "12/2021",
		SecurityCode = "999"
	},
	//DEMAIS DADOS DO OBJETO DA TRANSA��O, CONFORME O MODELO ACIMA
};
```

O m�todo `Debit` � o respons�vel pelo envio de uma transa��o com cart�o de d�bito e est� na classe `Checkout`:

```
var debit = new Transaction<DebitCard> { /*CORPO DO OBJETO DA TRANSA��O*/ };

var checkout = new Checkout(config);
var response = (CheckoutResponse)checkout.Debit(debit);

Console.WriteLine($"Transa��o {response.IdTransaction} gerada com sucesso!");

Console.WriteLine($"Pagamento pendente. Por favor, acesse a p�gina {response.AuthenticationUrl} para finalizar o pagamento atrav�s do Internet Banking de sua Institui��o Banc�ria!");
```

## Solicita��es de Cobran�a / Vendas R�pidas

Na classe `Invoice` est�o os m�todos dispon�veis pela gera��o e tratamento de solicita��es de cobran�a.

O objeto para uma nova venda deve ser montado seguindo o modelo abaixo:

```
var singleSale = new SingleSale
{
    Customer = new Customer
    {
        Name = "Destinat�rio da Cobran�a",
        Identity = "01579286000174", //CPF ou CNPJ do Destinat�rio
        Email = "email@empresa.com.br",
        Address = new Address
        {
            Street = "Endere�o do Destinat�rio",
            Number = "N�mero 123",
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
    ExpirationDate = new DateTime(2019, 07, 10), //Data de expira��o da cobran�a
    DiscountAmount = 10, //Valor do desconto, em Reais, caso exista
    Instruction = "Solicita��o de cobran�a pelos produtos 1, 2 e 3.",
    PaymentMethods = new List<PaymentMethod>
    {
        new PaymentMethod { CodePaymentMethod = "1" }, //Boleto
        new PaymentMethod { CodePaymentMethod = "2" }, //Cr�dito
        new PaymentMethod { CodePaymentMethod = "3" }, //Bitcoin
        new PaymentMethod { CodePaymentMethod = "4" }, //D�bito
    },
    DueDate = new DateTime(2019, 07, 10), //Vencimento do Boleto Banc�rio, caso  habilitado
    InterestAmount = 2, //Valor dos Juros do Boleto Banc�rio, caso  habilitado
    PenaltyAmount = 3, //Valor da Multa do Boleto Banc�rio, caso  habilitado
    Messages = new List<string> { "Mensagem 1", "Mensagem 2" },
    Emails = new List<string> { "email1@empresa.com.br", "email2@company.com" } //E-mails para envio da cobran�a
};

var invoice = new Invoice(config);
var response = (InvoiceResponse)invoice.New(singleSale);

Console.WriteLine($"Solicita��o {response.SingleSaleHash} gerada com sucesso!");

Console.WriteLine($"Siga o link para realizar o pagamento: {response.SingleSaleUrl} ");
```

Para realizar o cancelamento de uma solicita��o de cobran�a, basta realizar uma chamada para o m�todo `Cancel`, informando o hash gerado para a solicita��o desejada. A resposta ser� um booleano com a confirma��o do cancelamento.

```
//Utilizando a resposta do m�todo anterior como vari�vel
var saleToCancel = new SingleSale { SingleSaleHash = response.SingleSaleHash }; 

var invoice = new Invoice(config);
var confirmation = invoice.Cancel(saleToCancel);

if (confirmation) Console.WriteLine("Cobran�a cancelada com sucesso!");
```

## Informa��es adicionais / Contato

A orienta��o sobre a utiliza��o da API tamb�m est� dispon�vel na documenta��o de refer�ncia da API, [dispon�vel aqui]([https://developers.safe2pay.com.br/](https://developers.safe2pay.com.br/)), por�m salientamos que ela se encontra em atualiza��o para a nova vers�o da API e, por isso, recomendamos a utiliza��o do [pacote da galeria do NuGet](https://www.nuget.org/packages/Safe2Pay), para que voc� esteja sempre com a vers�o mais atualizada!

Em caso de d�vidas, [ficamos � disposi��o em nossos canais](https://safe2pay.com.br/contato) ou diretamente pelo e-mail <integracao@safe2pay.com.br>.