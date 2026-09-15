# MainDocs

Documentação vigente da plataforma Main: MainDDD, MainIntegration e produtos especializados.

## Ordem de leitura

1. [Plataforma de produtos dependentes do MainDDD](docs/architecture/plataforma-produtos-dependentes-mainddd.md) — papel do MainDDD, dos produtos especializados e do MainIntegration.
2. [Plano do provisionamento automático por cliente](docs/architecture/plano-provisionamento-automatico-por-cliente.md) — ordem das implementações, dependências, permissões e critérios de conclusão.
3. [Estrutura geral do MainDDD](docs/architecture/MainDDD%20estrutura%20geral.md) — solução, módulos, projetos, persistência, frontend e testes.
4. [Fundação DDD e CQRS](docs/architecture/ddd-foundation.md) — regras de domínio, comandos, consultas, handlers, dispatcher, pipeline e outbox.
5. [Caminho de uma requisição: leitura e escrita com agregado](docs/architecture/caminho-requisicoes-leitura-escrita.md) — aplicação concreta da arquitetura em Documentos.
6. [Tipos de saldo como classificadores de estoque](docs/estoque/tipos-saldo-classificadores.md) — regra de domínio para documentos, estoque e futuras integrações.
7. [Contratos protegidos da API](docs/architecture/api-contracts.md) — fronteira HTTP e como evoluí-la sem quebrar consumidores.
8. [Idempotência na criação de documentos](docs/idempotencia-documentos.md) — proteção das escritas contra repetição e conflito.
9. [Autenticação multitenant em desenvolvimento](docs/autenticacao-development.md) — tenant, usuário, papéis e testes locais.
10. [Plano de integração entre MainDDD, MainIntegration e MainPDV](docs/architecture/plano-integracao-mainddd-mainpdv.md) — sincronização de cadastros e venda integrada entre produtos.

## Manutenção

- Este repositório contém somente documentação vigente ou decisões arquiteturais ativas.
- A sequência acima parte do contexto de negócio, passa pela implementação interna e termina na integração entre produtos.
- Materiais de roadmap concluído, PRs históricas e interfaces substituídas não são mantidos aqui.
- Caminhos de código citados nos documentos são relativos à raiz do projeto correspondente.
