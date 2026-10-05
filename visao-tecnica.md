# Visão técnica e fundamentos

Este documento explica como os principais fundamentos de engenharia aparecem no código atual da plataforma Asha. Os exemplos são recortes representativos dos repositórios, não uma afirmação de que cada módulo segue todos os padrões de modo uniforme. A solução evoluiu em etapas: a Gestão contém tanto serviços de aplicação anteriores quanto o fluxo CQRS mais recente. Os links de código apontam para as branches de trabalho atuais verificadas durante esta revisão; eles podem mudar quando as alterações forem integradas.

## SOLID

SOLID é um conjunto de princípios para reduzir acoplamento e tornar mudanças localizadas. Há evidências concretas de vários princípios na solução; o grau de aplicação varia conforme o componente.

| Princípio | Como aparece | Exemplo verificável |
| --- | --- | --- |
| Responsabilidade única (SRP) | A regra de negócio, a coordenação do caso de uso, a persistência e a entrada HTTP ficam em tipos diferentes. | `Documento` protege invariantes do agregado; `CriarDocumentoHandler` encaminha o caso de uso; `DocumentoRepository` acessa dados; `DocumentoController` recebe HTTP. |
| Aberto/fechado (OCP) | Módulos registram seus handlers e implementações por uma entrada de composição, permitindo incluir comportamentos sem centralizar cada classe no host. | `AddDocumentosModule()` chama `AddModuleApplication(...)`, que descobre handlers no assembly do módulo. Novos módulos ainda exigem alterações explícitas no ponto de composição do host. |
| Substituição de Liskov (LSP) | Implementações podem substituir contratos quando preservam o comportamento esperado pelo consumidor. | Na Integração, transportes implementam `IIntegrationEventTransport`; o roteador de módulos usa essa abstração para encaminhar eventos sem depender diretamente do cliente RabbitMQ. O contrato precisa continuar sendo respeitado por cada transporte. |
| Segregação de interfaces (ISP) | Consumidores recebem contratos orientados a uma necessidade específica, em vez de uma interface única com todas as operações do sistema. | `IDocumentoQueryService` concentra consultas de documentos; `IDocumentoRepository` dá suporte ao acesso de domínio e escrita. |
| Inversão de dependência (DIP) | Casos de uso dependem de abstrações e contratos; detalhes de banco, mensageria e transporte ficam nas implementações. | Handlers recebem interfaces como `IDocumentoCommandService`, `IDocumentoRepository` e `IValidator<T>` pelo construtor. |

Isso é evidência de aplicação dos princípios, não uma certificação de conformidade integral. Por exemplo, o OCP é favorecido pela composição modular, mas adicionar um módulo ainda requer registrá-lo no host. A existência de interfaces, por si só, não garante DIP ou LSP: a direção das referências e a semântica do contrato também importam.

## DDD: conceitos e classes

Domain-Driven Design organiza o software em torno das regras do domínio, com uma linguagem compartilhada e limites claros entre conceitos. Na Gestão, o domínio de Documentos e os módulos ao redor usam os seguintes tipos:

| Classificação | Responsabilidade | Exemplo |
| --- | --- | --- |
| Entidade e raiz de agregado | Tem identidade própria e controla mudanças consistentes em si e nos seus filhos. | `Documento` é uma raiz de agregado. Métodos como `Criar`, `AdicionarItem`, `Processar`, `Cancelar` e `Estornar` guardam suas transições de estado e invariantes. |
| Entidade filha | Vive dentro do ciclo de vida do agregado que a contém e não é alterada por um caso de uso à parte. | `DocumentoItem` é criado e alterado pelo `Documento`; quantidade, preço e desconto são validados ao criar/alterar o item. |
| Value Object | Representa um valor definido por seus dados, com validação e sem identidade própria de negócio. | `PrefixoTipoDocumento` normaliza e valida um prefixo imutável; `Cpf` valida e conserva apenas os dígitos do CPF. |
| Serviço/política de domínio | Expressa uma regra do domínio que não pertence naturalmente a uma única entidade. | `PoliticaComercialDocumento` valida vigência da tabela de preço e condições comerciais de venda ou compra. |
| Evento de domínio | Registra um fato de negócio ocorrido no agregado, para reação interna desacoplada. | `Documento.Criar` levanta `DocumentoCriadoDomainEvent`; `Processar` levanta `DocumentoProcessadoDomainEvent`. |
| Comando e consulta | Descrevem uma intenção de escrita ou uma pergunta, respectivamente. São mensagens da camada Application, não entidades do domínio. | `CriarDocumentoCommand` e `ListarDocumentosQuery`. |
| Handler de aplicação | Executa um caso de uso, valida ou coordena dependências e retorna o resultado da aplicação. Não deve concentrar regra de negócio que pertence ao agregado. | `CriarDocumentoHandler` delega a operação ao serviço de escrita; `ListarDocumentosHandler` usa o contrato de consulta. |
| Repositório/porta de acesso | Abstrai a recuperação e persistência de agregados para o domínio/aplicação. | `IDocumentoRepository` é implementado por `DocumentoRepository` na infraestrutura. |
| DTO/contrato de transporte | Leva dados entre API, aplicação e integrações; não substitui entidade ou Value Object. | `DocumentoCreateDTO` e `DocumentoResponseDTO`. |

O relacionamento relevante no exemplo é: o controller recebe uma mensagem, o handler coordena o caso de uso, o agregado `Documento` protege as regras de estado e a infraestrutura persiste o resultado pelo repositório e unidade de trabalho. A configuração do EF Core vive fora do domínio. O domínio ainda contém algumas referências explícitas entre módulos e a aplicação usa persistência compartilhada; portanto, trata-se de um monólito modular em evolução, não de bounded contexts tecnicamente isolados em serviços e bancos independentes.

## CQRS e impacto no fluxo

Command Query Responsibility Segregation separa a intenção de alterar estado da intenção de consultar estado. Na Gestão, isso aparece em contratos, handlers e dispatchers próprios (`ICommand<T>`, `IQuery<T>`, `ICommandHandler<,>`, `IQueryHandler<,>` e `ISender`). Não é MediatR e não significa que leitura e escrita usem bancos diferentes.

### Escrita: criar documento

1. `DocumentoController` recebe e converte a solicitação para `CriarDocumentoCommand`.
2. `ISender.SendAsync` resolve o handler registrado para esse tipo.
3. `CriarDocumentoHandler` delega a coordenação ao serviço de escrita.
4. O fluxo carrega ou cria o agregado `Documento`, que verifica invariantes e altera seu estado por métodos explícitos.
5. Repositório e unidade de trabalho persistem a mudança; eventos de domínio podem acionar reações internas.

### Leitura: listar documentos

1. O controller envia `ListarDocumentosQuery` por `ISender.QueryAsync`.
2. `ListarDocumentosHandler` chama `IDocumentoQueryService`.
3. `DocumentoQueryService` compõe filtros, paginação e projeção para DTO. Nas consultas diretas usa `AsNoTracking` e `ProjectTo`/projeção, evitando carregar agregados rastreados quando a resposta é somente de leitura.

O efeito prático é deixar explícito o propósito do fluxo, organizar validação e autorização por caso de uso e permitir otimizar consultas sem enfraquecer as regras de escrita do agregado. Os dois lados ainda compartilham modelo relacional e infraestrutura de persistência. Há também serviços de comando mais antigos, como `DocumentoCommandService`, que continuam sendo usados por handlers enquanto operações são migradas. Por isso, CQRS está aplicado de forma lógica na aplicação, mas não corresponde a uma separação física completa de modelos, bancos ou deploys.

## Inversão de controle e injeção de dependência

Inversão de Controle (IoC) é o princípio de que o fluxo de criação e coordenação não fica sob controle direto do consumidor. Injeção de Dependência (DI) é uma das técnicas usadas para implementá-lo: o runtime fornece dependências ao construir o componente.

No .NET, o ponto de composição está no `Program.cs` e nos métodos `Add...Module()`. Eles registram implementações no `IServiceCollection`; handlers e serviços recebem suas dependências nos construtores. Por exemplo, `ListarDocumentosHandler(IDocumentoQueryService queries)` conhece o contrato, enquanto o módulo associa esse contrato à classe concreta `DocumentoQueryService`. O handler não cria o serviço com `new` nem conhece EF Core.

O `CqrsSender` é outro exemplo: recebe `IServiceProvider` para localizar o handler adequado e executa os comportamentos de pipeline registrados. Esse dispatcher é infraestrutura da aplicação; regras do domínio não devem depender dele. A composição explícita por módulo facilita identificar onde uma implementação entra e trocar detalhes externos sem propagar referências concretas pelas camadas internas.

## Bibliotecas e particularidades

As versões abaixo foram conferidas nos arquivos de projeto dos repositórios atuais. A plataforma tem runtime alvo .NET 8, mas versões de pacotes não são idênticas entre aplicativos; especialmente Entity Framework Core e Npgsql variam. Ao atualizar, validar cada `.csproj` e a compatibilidade do provedor correspondente.

| Biblioteca | Uso atual e exemplo |
| --- | --- |
| ASP.NET Core / Minimal APIs | Hospeda APIs, controllers e endpoints. No Portal, `PortalBrowserEndpoints` mapeia catálogo, antiforgery e envio de interesses. Um endpoint valida o pedido e delega a integração a uma abstração injetada. |
| Entity Framework Core + Npgsql | Persistência relacional PostgreSQL. Gestão e PDV declaram EF Core 9.0.9 e Npgsql 9.0.4; Identity declara EF Core 8.0.15 e Npgsql 8.0.11; Integration usa Npgsql 8.0.11. EF configura mapeamentos, consultas e unidade de trabalho; Npgsql é o provedor específico do PostgreSQL. |
| OpenIddict + ASP.NET Identity | Identity hospeda os fluxos OpenID Connect/OAuth2, usuários, sessões e emissão de tokens. `AuthorizationController` valida a sessão e os escopos; `UserTokenService` determina claims e destinos no token. Pacotes OpenIddict 7.7.1; Identity EF Core 8.0.15. |
| RabbitMQ.Client | Integração usa `RabbitMqEventTransport` para publicar eventos persistentes em exchange topic e definir routing key por módulo/tipo. É a biblioteca de transporte AMQP, não a implementação do padrão Outbox. Versão 7.1.2. |
| EF Core Outbox/Inbox (implementação própria) | `EfIntegrationOutbox` grava mensagem antes do envio; `OutboxDispatcher` publica pendências e marca sucesso/erro; `EfIntegrationInbox` detecta `EventId` repetido para evitar reprocessar duplicatas. O desenho combina persistência no PostgreSQL com transporte RabbitMQ. |
| HttpClientFactory | Portal registra clientes tipados para catálogo, pedido e provisionamento em `AddPortalInfrastructure`. `HttpInterestLeadSender` usa `PostAsJsonAsync` para encaminhar a intenção ao contrato do Integration. O Identity também é consultado por clientes de serviço nos demais aplicativos. |
| FluentValidation | Gestão e PDV declaram FluentValidation 11.3.0 para validar entradas antes da execução dos casos de uso. Exemplo: `DocumentoCreateDTOValidator` valida dados de entrada; as invariantes finais continuam protegidas pelo domínio. |
| AutoMapper | Gestão e PDV usam AutoMapper.Extensions.Microsoft.DependencyInjection 12.0.0 para transformar modelos e DTOs; `ProjectTo` também permite projetar consulta no banco quando a expressão suporta essa projeção. Não deve mover regra de negócio para perfis de mapeamento. |
| Serilog | Gestão e PDV registram logs estruturados, com sinks de Console 6.0.0 e Seq 9.0.0. É útil correlacionar falhas de requisições e processamento sem acoplar domínio a um destino específico de log. |
| Microsoft Fluent UI Blazor | Portal, Gestão e PDV usam componentes visuais compartilhados da biblioteca Fluent UI Blazor, na versão 4.14.4, para controles e layout da interface. |
| Swashbuckle | Publica documentação OpenAPI/Swagger para APIs. Gestão/PDV declaram 9.0.5; Portal API e Integration declaram 6.6.2. Os exemplos de configuração devem acompanhar a versão do projeto. |
| MailKit / MimeKit | Identity usa MailKit 4.17.0 para transporte SMTP e MimeKit para montar mensagens. O segredo SMTP fica em configuração protegida, não no código. |
| QRCoder | Identity declara QRCoder 1.8.0 para produzir o QR de configuração do autenticador TOTP durante habilitação de segundo fator. |
| xUnit e ASP.NET Core MVC Testing | Testes automatizados usam xUnit; os projetos de teste de API usam `Microsoft.AspNetCore.Mvc.Testing` para hospedar a aplicação em testes de integração/contrato. As versões variam por repositório. |

### Exemplo integrado: cadastro de interesse no Portal

O fluxo mostra como as bibliotecas e os padrões se combinam sem colocar as regras em um único arquivo:

```csharp
endpoints.MapPost("/interesses", async (
    HttpContext context,
    IAntiforgery antiforgery,
    InterestLeadRequest request,
    IPortalProductCatalog catalog,
    IInterestLeadSender sender,
    CancellationToken ct) =>
{
    await antiforgery.ValidateRequestAsync(context);
    var validCodes = (await catalog.ListAsync(ct))
        .Select(product => product.Codigo)
        .ToHashSet(StringComparer.OrdinalIgnoreCase);

    if (request.ProductCodes.Any(code => !validCodes.Contains(code)))
        return Results.BadRequest();

    return Results.Ok(await sender.SendAsync(request, ct));
});
```

O endpoint ASP.NET Core protege a requisição contra CSRF com antiforgery, valida se os produtos selecionados pertencem ao catálogo obtido pela abstração `IPortalProductCatalog` e delega o envio a `IInterestLeadSender`. A implementação `HttpInterestLeadSender` usa `HttpClient` e serialização JSON para chamar o contrato publicado pelo Integration. Assim, a interface não conhece a implementação HTTP nem o banco de Gestão.


Leia também a [Aplicações](aplicacoes.md) e a [Infraestrutura](infraestrutura.md). Os códigos podem evoluir; os links apontam para as implementações nos repositórios Asha.

## Asha Gestão

| Área | Responsabilidade |
| --- | --- |
| src/AshaAPI | Host ASP.NET Core, HTTP, autenticação e composição. |
| src/Modules | Catálogo, Pessoas, Precificação, Documentos, Estoque, IdentidadeAcesso e Administração. |
| Camadas dos módulos | Bibliotecas Domain, Application, Infrastructure e Contracts para regras, casos de uso, implementações e comunicação. |
| src/BuildingBlocks | Abstrações compartilhadas de domínio e aplicação. |
| src/Core | Código compartilhado que convive com a modularização. |
| src/Persistence | Persistência, DbContext, mapeamentos e migrações. |
| Frontend/Asha/Asha.Client | Interface Blazor WebAssembly. |

O monólito modular compartilha host e banco, permitindo coordenar transações internas. Contratos HTTP e eventos formam a fronteira com outros produtos.

## Asha Ponto de Venda

| Área | Responsabilidade |
| --- | --- |
| Projeto de API na raiz | Host e composição do backend. |
| Modules | Venda, Caixa, Documentos, Estoque, Fiscal, PosVenda, Pessoas, Precificacao, CatalogoLocal, IdentidadeAcesso, Administracao, Configuracao e Integracao. |
| BuildingBlocks | Entidades, agregados, eventos, comandos, consultas, erros, IUnitOfWork e IClock. |
| Frontend/Asha/Asha | Host web da interface e comunicação com a API. |
| Frontend/Asha/Asha.Client | Interface Blazor WebAssembly. |
| Tests | Testes de regras e contratos HTTP. |

Os módulos organizam código dentro do projeto de API, diferentemente das bibliotecas separadas da Gestão. A fundação DDD é incremental. AppDbContext implementa a unidade de trabalho e um middleware traduz erros de domínio para a fronteira HTTP.

## Asha Portal

| Projeto em src | Responsabilidade |
| --- | --- |
| AshaPortal.Domain | Modelos da entrada comercial e preparação. |
| AshaPortal.Application | Contratos e coordenação de catálogo, interesse e onboarding. |
| AshaPortal.Infrastructure | Adaptadores HTTP e acesso aos serviços. |
| AshaPortal.WebAPI | Host da API. |
| AshaPortal.Web | Host web, endpoints do navegador e composição. |
| AshaPortal.Web.Client | Interface Blazor WebAssembly. |
| tests/AshaPortal.UnitTests | Testes de comportamentos e contratos. |

Camadas e adaptadores mantêm autenticação de serviço e transporte fora da interface. O envio comercial passa pela Integração antes de chegar à Gestão; o Portal não consulta diretamente os bancos operacionais.

## Asha Identity

| Projeto ou área | Responsabilidade |
| --- | --- |
| src/AshaIdentity.Web | Host, ASP.NET Core Identity e OpenIddict. |
| Controllers e Pages | Endpoints, telas de conta e administração. |
| Data | DbContext, usuários e persistência de identidade. |
| Security e Services | Políticas, tokens, convites e serviços auxiliares. |
| src/AshaIdentity.Contracts | Modelos de comunicação. |
| src/AshaIdentity.Client | Recursos de cliente para uso da identidade. |
| tests/AshaIdentity.SecurityTests | Testes de autenticação e autorização. |

A implementação se concentra no host web e em serviços especializados, sem repetir a divisão de bibliotecas da Gestão. O Identity fornece identidade técnica; direitos de negócio são avaliados pelas aplicações. Mudanças de papéis técnicos e sua auditoria são persistidas na mesma transação.

## Asha Integração

| Projeto em src | Responsabilidade |
| --- | --- |
| AshaIntegration.Domain | Modelo dos eventos e estados. |
| AshaIntegration.Application | Abstrações de recebimento, publicação e entrega. |
| AshaIntegration.Infrastructure | PostgreSQL/EF Core, Inbox, Outbox, RabbitMQ, gateways HTTP e identidade de serviço. |
| AshaIntegration.Api | Endpoints de eventos, pedidos e administração; composição. |
| AshaIntegration.Worker | Host de processamento em segundo plano disponível no código. |
| AshaIntegration.Frontend | Administração em Blazor WebAssembly. |
| tests/AshaIntegration.UnitTests | Testes de regras e contratos. |

Adaptadores de transporte e persistência mediam aplicações. Pedidos comerciais usam encaminhamento HTTP; eventos usam recepção, despacho e entrega. Nem toda operação passa por RabbitMQ.

A API registra hosted services de consumo e despacho do Outbox. A stack básica publica API e frontend; a existência do projeto Worker não significa sua implantação como serviço separado.

## Asha Onboarding & Deploy

| Projeto ou área | Responsabilidade |
| --- | --- |
| src/AshaProvisioning.Domain | Solicitações, operações e estados. |
| src/AshaProvisioning.Application | Casos de uso, repositórios e abstrações de etapas. |
| src/AshaProvisioning.Contracts | Pedidos e respostas. |
| src/AshaProvisioning.Infrastructure | Persistência e etapas locais ou externas. |
| src/AshaProvisioning.Api | Recepção e consulta de solicitações. |
| src/AshaProvisioning.Worker | Busca operações prontas e executa etapas habilitadas. |
| deploy, config, scripts e workflows | Dockerfiles, revisões e automação de publicação. |
| tests/AshaProvisioning.UnitTests | Testes de regras e preparação. |

O recebimento HTTP é separado do processamento demorado. As etapas permitem acompanhar estados e tentativas sem concentrar a preparação no controller. Há persistência relacional e em memória; a referência usa PostgreSQL. O provisionamento dedicado por cliente permanece uma evolução futura.

Parte dos diretórios mantém AshaProvisioning ou AshaIntegration, embora assemblies e soluções usem nomes atuais dos produtos. Pipeline de onboarding e automação de deploy compartilham repositório, mas têm responsabilidades diferentes.

## Executor e documentação

O asha_deployExec contém scripts de instalação, configuração de exemplo e workflow de validação. Prepara o runner oficial; a construção e publicação das aplicações pertencem ao Onboarding & Deploy.

O Asha-Docs contém quatro arquivos Markdown: README, Infraestrutura, Aplicações e Visão Técnica.

### Código-fonte citado

- [Gestão: agregado Documento](https://github.com/JAyres88/Asha-Gestao/blob/codex/gestao-pedido-portal-produtos-20261001/src/Modules/Documentos/Domain/Entities/Documento.cs)
- [Gestão: DocumentoItem](https://github.com/JAyres88/Asha-Gestao/blob/codex/gestao-pedido-portal-produtos-20261001/src/Modules/Documentos/Domain/Entities/DocumentoItem.cs)
- [Gestão: Value Objects CPF e Prefixo](https://github.com/JAyres88/Asha-Gestao/tree/codex/gestao-pedido-portal-produtos-20261001/src/BuildingBlocks/Domain/Shared/ValueObjects)
- [Gestão: handlers de escrita](https://github.com/JAyres88/Asha-Gestao/blob/codex/gestao-pedido-portal-produtos-20261001/src/Modules/Documentos/Application/Documentos/DocumentoWriteMessages.cs)
- [Gestão: handlers de leitura](https://github.com/JAyres88/Asha-Gestao/blob/codex/gestao-pedido-portal-produtos-20261001/src/Modules/Documentos/Application/Documentos/DocumentoReadMessages.cs)
- [Gestão: sender CQRS](https://github.com/JAyres88/Asha-Gestao/blob/codex/gestao-pedido-portal-produtos-20261001/src/AshaAPI/Infrastructure/Shared/Services/CqrsSender.cs)
- [Gestão: composição do aplicativo](https://github.com/JAyres88/Asha-Gestao/blob/codex/gestao-pedido-portal-produtos-20261001/src/AshaAPI/Infrastructure/Shared/Services/ApplicationService.cs)
- [Portal: endpoints do navegador](https://github.com/JAyres88/Asha-Portal/blob/codex/portal-pedido-venda-20261001/src/AshaPortal.Web/Endpoints/PortalBrowserEndpoints.cs)
- [Portal: encaminhamento de interesse por HTTP](https://github.com/JAyres88/Asha-Portal/blob/codex/portal-pedido-venda-20261001/src/AshaPortal.Infrastructure/Commercial/HttpInterestLeadSender.cs)
- [Identity: autorização OpenID Connect](https://github.com/JAyres88/Asha-Identity/blob/codex/identity-portal-order-scope-20261001/src/AshaIdentity.Web/Controllers/AuthorizationController.cs)
- [Integration: transporte RabbitMQ](https://github.com/JAyres88/Asha-Integracao/blob/codex/integracao-pedido-portal-20261001/src/AshaIntegration.Infrastructure/Messaging/RabbitMqEventTransport.cs)
- [Integration: Outbox e Inbox](https://github.com/JAyres88/Asha-Integracao/tree/codex/integracao-pedido-portal-20261001/src/AshaIntegration.Infrastructure/Messaging)
