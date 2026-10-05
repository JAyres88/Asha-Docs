# Infraestrutura

Descrição das tecnologias e da configuração de referência da plataforma Asha, verificada no código e nos manifestos em 5 de outubro de 2026.

## Tecnologias e componentes de apoio

| Tecnologia ou componente | Uso |
| --- | --- |
| C# e .NET 8 | Base das aplicações, APIs e processos em segundo plano. |
| ASP.NET Core | APIs HTTP, autenticação, autorização e hospedagem web. |
| Blazor WebAssembly | Interfaces de Gestão, PDV, Portal e administração da Integração. |
| Razor Pages e ASP.NET Core Identity | Telas de conta e administração de identidade. |
| Microsoft Fluent UI | Componentes visuais em projetos como Portal e Gestão. |
| Entity Framework Core e Npgsql | Mapeamento relacional, PostgreSQL e migrações. A versão do EF Core varia entre projetos. |
| OpenIddict | Implementação de OpenID Connect e OAuth 2.0 no Identity. |
| FluentValidation e AutoMapper | Validação e conversão de modelos nos projetos que adotam essas bibliotecas. |
| Swagger/OpenAPI e Swashbuckle | Descrição e exploração dos contratos HTTP. |
| Serilog | Logs estruturados em projetos como Gestão, com suporte a console e ao destino Seq. |
| MailKit e MimeKit | Envio por SMTP no Identity, com provedor configurado externamente. |
| xUnit | Testes de regras, contratos e segurança, conforme o repositório. |
| PowerShell e GitHub Actions | Preparação, validação e publicação do ambiente. |

Uma dependência de biblioteca não implica serviço implantado. Por exemplo, a stack de referência não define servidor Seq; seu uso depende de configuração adicional.

## Hospedagem e publicação web

A referência local usa Docker Desktop com contêineres Linux e Docker Swarm. O Swarm administra serviços, réplicas, redes, volumes e configurações. Um registry local armazena as imagens versionadas da Asha.

| Componente | Função |
| --- | --- |
| Docker Swarm | Execução e atualização dos serviços. |
| Registry local | Armazenamento das imagens construídas pela esteira. |
| Traefik 3.6 | Gateway que encaminha tráfego conforme hostname e rota. |
| Cloudflare DNS e Tunnel | Publicação HTTPS por conexão de saída até o gateway. |
| Nginx | Servidor estático na imagem do frontend da Integração. |
| GitHub Actions Runner | Execução dos trabalhos autorizados na máquina da instalação. |

O caminho externo é navegador → Cloudflare → túnel → Traefik → aplicação. O GitHub armazena código e coordena automação; as aplicações continuam executando no host da instalação. O ambiente depende da disponibilidade da máquina, do Docker e do túnel.

A rede interna main-provider conecta aplicações, banco e broker. O manifesto básico define acessos locais; os scripts de túnel geram rotas externas. A publicação da Integração pode reunir frontend e caminhos administrativos em um hostname, mantendo endpoints de eventos e comunicação interna fora dessa exposição.

## Banco de dados

O manifesto atual usa **PostgreSQL 16**, com uma instância e bancos separados por aplicação. A documentação anterior mencionava SQL Server; a configuração atual substitui essa referência. Isso não comprova a migração de todas as instalações existentes.

| Banco de referência | Conteúdo |
| --- | --- |
| asha_identity | Usuários, papéis, clientes e dados OAuth/OIDC. |
| asha_gestao | Cadastros compartilhados, documentos, estoque, administração e acesso. |
| asha_pdv | Dados da operação do ponto de venda. |
| asha_integration | Recepção, publicação e acompanhamento de entregas. |
| asha_provisioning | Solicitações e estados de preparação técnica. |

A stack não configura banco relacional exclusivo para o Portal: ele encaminha operações aos serviços responsáveis. Os módulos da Gestão compartilham a persistência dessa aplicação. Aplicações distintas se comunicam por contratos HTTP e mensagens.

O acesso relacional usa EF Core com Npgsql. Migrações versionadas acompanham mudanças de esquema. PostgreSQL e RabbitMQ possuem volumes persistentes; atualizar imagens ou reduzir réplicas não elimina esses dados. O Identity também mantém volume para material de proteção de dados.

## Mensageria e comunicação

O broker é **RabbitMQ 4.1**, na imagem que inclui o complemento de administração. A Integração utiliza RabbitMQ.Client. Esse complemento não implica acesso público ao broker.

- **HTTP:** consultas e comandos com resposta imediata, incluindo pedidos comerciais do Portal.
- **Inbox:** registro de recepção de eventos e identificação de duplicatas.
- **Outbox:** persistência de publicações pendentes para envio posterior.
- **Entrega:** contratos versionados, tentativas e tratamento de falhas, incluindo dead letter.

Esses mecanismos reduzem perdas e efeitos de reentrega, sem garantir execução única de qualquer operação. Os destinos também precisam aplicar as regras de idempotência dos contratos.

## Identidade e configuração

O Identity emite tokens OIDC/OAuth 2.0. Interfaces usam authorization code com PKCE; serviços usam client credentials. APIs validam audiência, escopos e permissões. O Identity autentica, a Gestão fornece contexto global de acesso e cada aplicação autoriza suas operações.

A administração técnica da Integração exige IntegrationAdministrator. Ser administrador da Gestão ou do Identity não concede esse papel automaticamente.

Senhas, chaves, credenciais de banco, segredos de clientes e configuração SMTP são fornecidos por configuração privada e variáveis de execução. Arquivos públicos do frontend contêm apenas parâmetros necessários ao navegador. Identificadores legados main*, como audiências, escopos e recursos persistidos, são contratos de compatibilidade e não devem ser alterados apenas para acompanhar a marca.

## Implantação e operação

Cada aplicação mantém sua CI. O Onboarding & Deploy centraliza revisões aprovadas, construção das imagens, atualização do Swarm e verificação. O asha_deployExec prepara a máquina e configura o runner oficial do GitHub Actions.

A esteira seleciona revisões explícitas e verifica migrações e aprovação do esquema. A existência de scripts de migração não significa aplicação automática em todo deploy. A recuperação de imagens também não reverte dados; backup e recuperação do banco precisam de tratamento próprio.

O manifesto básico mantém o envio de provisionamento desabilitado e o worker de onboarding com zero réplicas. Sua ativação depende de configuração. A criação automática de infraestrutura dedicada por cliente não está concluída.

## Referências

- [Manifesto de implantação](https://github.com/JAyres88/Asha-Onboarding-Deploy/blob/main/deploy/stack.yml).
- [Onboarding & Deploy](https://github.com/JAyres88/Asha-Onboarding-Deploy).
- [Executor de implantação](https://github.com/JAyres88/asha_deployExec).
- [Identity](https://github.com/JAyres88/Asha-Identity) e [Integração](https://github.com/JAyres88/Asha-Integracao).

Veja também [Aplicações](aplicacoes.md) e [Visão Técnica](visao-tecnica.md).