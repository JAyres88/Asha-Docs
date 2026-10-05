# Visão Técnica

Padrões e macroestrutura dos projetos Asha verificados em 5 de outubro de 2026. A organização varia entre aplicações e parte da adoção é incremental, convivendo com código anterior.

## Padrões de projeto e arquitetura

| Padrão ou prática | Aplicação |
| --- | --- |
| Monólito modular | Gestão reúne módulos em uma aplicação com bibliotecas próprias; PDV organiza áreas de negócio no mesmo projeto de API. |
| Camadas | Domain contém modelo e regras; Application coordena casos de uso; Infrastructure implementa persistência e integrações; hosts expõem HTTP, interface ou processamento. |
| DDD incremental | Gestão e PDV adotam entidades, agregados, eventos e regras de domínio, com cobertura diferente entre módulos. |
| Injeção de dependências | Hosts registram implementações para as abstrações dos casos de uso. |
| Repository e Unit of Work | Isolam acesso e confirmação da persistência onde adotados, com implementação relacional por EF Core/DbContext. |
| Comandos e consultas | A fundação do PDV distingue alterações de estado de leituras; isso não implica bancos separados ou Event Sourcing. |
| DTOs e contratos | Separam os modelos de comunicação das entidades persistidas. |
| Adaptadores e gateways | Encapsulam transporte HTTP e acesso a serviços externos. |
| Inbox, Outbox e idempotência | Integração registra recepção e publicações pendentes, identifica duplicatas e acompanha reentregas. |
| Hosted services e workers | Executam despacho de mensagens e preparação fora das requisições de usuário. |
| Pipeline de etapas | Onboarding coordena implementações de IProvisioningStep, com estados, tentativas e retomada. |
| Políticas de autorização | Protegem operações por audiência, escopos, papéis e contexto de acesso. |

Os módulos internos não são todos microsserviços. Gestão e PDV mantêm organização modular dentro de suas fronteiras de execução.

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

## Referências

As estruturas pertencem a [Gestão](https://github.com/JAyres88/Asha-Gestao), [PDV](https://github.com/JAyres88/Asha-Ponto-de-Venda), [Portal](https://github.com/JAyres88/Asha-Portal), [Identity](https://github.com/JAyres88/Asha-Identity), [Integração](https://github.com/JAyres88/Asha-Integracao), [Onboarding & Deploy](https://github.com/JAyres88/Asha-Onboarding-Deploy) e [deployExec](https://github.com/JAyres88/asha_deployExec).

Veja também [Infraestrutura](infraestrutura.md) e [Aplicações](aplicacoes.md).