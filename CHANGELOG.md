## Changelog

Todas as mudan�as e atualiza��es relevantes ser�o listadas neste documento, assim como uma previs�o das funcionalidades que ser�o implementadas em novas vers�es.

#### Planejado para a pr�xima vers�o

- [ ] **Exemplos de uso de todos os m�todos dispon�veis na biblioteca**;
- [ ] Refinamento no tratamento das respostas de todos os m�todos;
- [ ] Tratamento customizado de exce��es;
- [ ] Tratamento para quando a resposta excede o tempo limite de timeout.

#### [1.0.8] - 26, Jul, 2019

- Adicionada a classe `RetryHandler`, com a a utiliza��o da biblioteca Polly, finalidade de tornar o HttpClient mais resiliente e ter um melhor recurso para o tratamento de falhas das chamadas.

#### [1.0.7] - 26, Jul, 2019

- Ajustes de nomenclatura de classes e m�todos para fins de padroniza��o;
- Inclus�o de classes para gest�o de Planos (`PlanRequest` e `PlanResponse`) e Ades�es (`SubscriptionRequest` e `SubscriptionResponse`);
- Separa��o do m�todo de tokeniza��o de dados de cart�o de cr�dito para a classe `TokenRequest`;
- Inclus�o do m�todo `Resend` em `InvoiceRequest` para o reenvio de solicita��es de cobran�a;
- Inclus�o do m�todo `ListByReference` em `TransactionRequest` para o consulta de transa��es pela refer�ncia;
- Pequenos ajustes.

#### [1.0.4 - 1.0.6] - 16, Jul, 2019

- Retirada da necessidade do envio da propriedade `IsSandbox`  das configura��es;
- Padroniza��es em geral, namespaces corrigidos e defini��es;
- Inclus�o da classe `Transfer`, para gest�o de transfer�ncias banc�rias.

#### [1.0.2 - 1.0.3] - 12, Jul, 2019

- Remo��o dos m�todos est�ticos;
- `Account` para intera��es relacionadas � conta-corrente e dados banc�rios;
- `Marketplace` para gest�o de subcontas;
- `Sales`, de gest�o de solicita��es de cobran�a, renomeada para `Invoice`  para fins de padroniza��o;
- `Transactions`, para intera��es com transa��es, renomeada para `Transaction`.

#### [1.0.0 - 1.0.1] - 9, Jul, 2019

- Vers�o inicial da Biblioteca de Integra��o em C#;
- `Config` para os dados de autentica��o na API e do ambiente, se Sandbox ou Produ��o;
- M�todos para a gera��o de transa��es dispon�veis na classe `Checkout`;
- M�todos para gest�o de solicita��es de cobran�a em `Sales`;
- M�todos para intera��es com transa��es em `Transactions`.