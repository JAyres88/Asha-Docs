# Roadmap de transposição para CQRS

## Objetivo e limites

Migrar MainDDD e, depois, MainPDV para CQRS sem reescrever o domínio, alterar contratos HTTP ou
interromper o frontend. A transposição será incremental: endpoints migrados passam a usar comandos
ou consultas; os demais continuam temporariamente nos UseCases.

CQRS aqui separa responsabilidades e modelos de leitura/escrita. Não implica dois bancos, event
sourcing ou microsserviços; essas decisões exigem requisitos próprios.

## Arquitetura de destino: module-first

```text
MainAPI (host HTTP / composition root)
BuildingBlocks (abstrações neutras compartilhadas)
Modules/
  Pessoas/{Domain, Application, Infrastructure, Presentation}
  Catalogo/{Domain, Application, Infrastructure, Presentation}
  Precificacao/{Domain, Application, Infrastructure, Presentation}
  Documentos/{Domain, Application, Infrastructure, Presentation}
  Estoque/{Domain, Application, Infrastructure, Presentation}
  IdentidadeAcesso/{Domain, Application, Infrastructure, Presentation}
  Administracao/{Application, Infrastructure, Presentation}
Main.Client -> contratos HTTP do MainAPI
```

- cada módulo reúne todas as camadas e especificidades de seu contexto;
- o Domain de um módulo não referencia Application, Infrastructure, ASP.NET, EF ou Swagger;
- Application referencia o Domain do próprio módulo e BuildingBlocks;
- Infrastructure implementa as portas do próprio módulo;
- MainAPI apenas recebe HTTP, autoriza, envia a mensagem e traduz o resultado;
- comandos alteram estado; consultas nunca alteram estado;
- controllers não acessam DbContext, repositórios ou UseCases;
- o frontend permanece desacoplado da organização interna do backend.

## BuildingBlocks

`BuildingBlocks` abstrai somente mecanismos comuns: `Entity`, `AggregateRoot`, eventos, Guard,
Result, mensagens CQRS, behaviors, relógio, transação, correlação e infraestrutura de Outbox. Ele
não contém Pessoa, Produto, Documento, Estoque, DTOs ou regras comerciais. Módulos dependem de
BuildingBlocks; BuildingBlocks nunca depende de módulos.

## Convenção de módulo e funcionalidade

```text
Modules/Catalogo/
  Domain/{Entities, ValueObjects, Events, Services, Specifications}
  Application/Produtos/
    Commands/CriarProduto/{Command, Handler, Validator}
    Queries/ListarProdutos/{Query, Handler}
  Infrastructure/{Persistence, Repositories, Queries}
  Presentation/{Controllers, Contracts}
```

Cada mensagem representa uma intenção de negócio. Handlers genéricos de CRUD serão evitados porque
esconderiam regras específicas e apenas recriariam os UseCases com outro nome.

## MainDDD

### PR 91 — Fundação e fluxo-piloto

- criar `MainAPI.Domain` e `MainAPI.Application`;
- introduzir mensagens, handlers e `ISender`;
- remover Swagger do Domain;
- migrar Categoria como prova vertical;
- preservar contratos HTTP e testes.

### PR 92 — BuildingBlocks e modelo de módulo

- extrair Common, Messaging e Utilities para projetos BuildingBlocks;
- criar o esqueleto module-first com as quatro camadas internas;
- adicionar testes de arquitetura para as novas fronteiras;
- documentar contratos permitidos entre módulos;
- manter assemblies globais atuais apenas como estrutura transitória.

### PR 93 — Módulo Catálogo e pipeline CQRS

- validação automática antes dos handlers;
- tratamento uniforme de resultados e exceções;
- transação apenas em comandos que exigem atomicidade;
- logging, correlação e duração das mensagens;
- dispatch tipado/registrado e testes dos comportamentos.
- mover Produto, Categoria, Embalagem e associações para `Modules/Catalogo`;
- concluir Categoria sem delegação ao UseCase legado;
- migrar todos os endpoints do contexto.

### PR 94 — Módulo Pessoas

- commands de Pessoa, Endereço e vínculos PessoaEndereco;
- queries de lista, detalhe e endereços vinculados;
- preservar endereço avulso e compartilhado;
- remover os UseCases desses controllers.

### PR 95 — Módulo Precificação

- commands de TabelaPreco e ProdutoPreco;
- queries de vigência, apresentação e preços associados;
- validar conflitos de período na escrita;
- manter seleção dinâmica de tabela e preço.

### PR 96 — Fundação do módulo Documentos

- TipoDocumento, versões e regras de saldo;
- Local, TipoSaldo e LocalTipoSaldo;
- NumeradorDocumento como porta transacional;
- associações por commands e grids por queries;
- preservar concorrência otimista.

### PR 97 — Escrita de Documentos

- commands separados para criar, atualizar, processar, cancelar e estornar;
- manter o agregado como autoridade das invariantes;
- preservar idempotência, numeração, política comercial e transação;
- eventos e Outbox na mesma transação;
- preservar endereço declarado e associação opcional à Pessoa.

### PR 98 — Leitura de Documentos

- queries de lista, detalhe, itens e saldos;
- projeções sem rastreamento e paginação no banco;
- filtros e exportação sem carregar agregados para exibição;
- manter os contratos usados pelos grids.

### PR 99 — Módulo Estoque

- commands de entrada, saída e movimento originado por documento;
- queries de saldo e histórico;
- concorrência otimista por saldo e idempotência por correlação;
- impedir alteração direta fora do fluxo de domínio.

### PR 100 — Módulo Administração

- queries de Auditoria e Outbox;
- commands de reprocessamento;
- Importação dividida entre análise (query) e aplicação (command);
- Identidade Visual por query/command;
- background workers usando portas de Application.

### PR 101 — Módulo Identidade e Acesso

- manter autenticação como infraestrutura;
- identidade do usuário exposta por porta de Application;
- autorização no endpoint e regra de negócio no handler;
- preservar modo DEV e preparar Entra ID sem tenant comercial desnecessário.

### PR 102 — Infrastructure compartilhada e composition root

- consolidar acesso ao banco, migrations, segurança, mensageria e integrações técnicas;
- cada configuração EF permanece pertencendo ao módulo dono da entidade;
- o host apenas registra módulos, middleware e endpoints;
- preservar o contexto e os comandos de migrations.

### PR 103 — Remoção da arquitetura transitória

- excluir UseCases e query services substituídos;
- remover interfaces duplicadas e a dependência transitória de EF em Application;
- separar testes por Domain, Application, Infrastructure e Contract;
- adicionar testes de arquitetura contra referências inválidas;
- atualizar documentação e comandos de migrations.

## MainPDV

O MainPDV começa após a PR 103, reutilizando o BuildingBlocks e o formato module-first comprovado,
mas preservando seu domínio e ciclo de deploy independentes.

### PR 104 — Fundação CQRS no MainPDV

- criar módulos Configuracao, CatalogoLocal, Venda, Caixa, Fiscal e PosVenda;
- cada módulo contém Domain, Application, Infrastructure e Presentation;
- reutilizar o desenho do pipeline sem copiar regras do MainDDD;
- converter Configuração do PDV como fluxo-piloto.

### PR 105 — Catálogo e réplica local

- queries de produto, embalagem, código de barras, preço e saldo;
- commands de carga física/importação e sincronização;
- checkpoints e disponibilidade do ERP fora da venda.

### PR 106 — Operação de venda

- commands de abrir venda, itens, desconto e consumidor;
- query otimizada da operação corrente para a tela única;
- processamento com regras, idempotência e concorrência.

### PR 107 — Caixa e pagamentos

- commands de abertura, suprimento, sangria e fechamento;
- queries da sessão e relatório gerencial;
- Strategy/Adapter para pagamentos e conciliação separada.

### PR 108 — Fiscal e contingência

- commands de emissão e cancelamento fiscal;
- Strategy por modo fiscal da instância;
- queries de situação e pendências;
- retentativa, contingência e deduplicação.

### PR 109 — Pós-venda

- cancelamento, estorno, devolução e troca;
- venda original imutável e vínculos explícitos;
- recomposição de estoque e acertos financeiros;
- reflexos fiscais assíncronos e auditáveis.

### PR 110 — Integração e encerramento

- eventos canônicos para MainIntegration/RabbitMQ;
- Outbox/Inbox e reprocessamento;
- remoção dos serviços legados;
- testes de arquitetura e documentação final.

## Política de transição

- uma funcionalidade migra inteira: endpoint, mensagem, handler e testes;
- código novo não chama UseCase legado, salvo o piloto da PR 91;
- legado só é removido quando nenhum endpoint o utiliza;
- CQRS não altera schema sem requisito funcional independente;
- mudança de contrato HTTP exige PR própria e coordenação com o frontend;
- cada branch nasce da `DEV` após o merge da predecessora;
- cada PR termina com build limpo e suítes unitária e contratual aprovadas.

## Definição de conclusão

Todos os endpoints usam `ISender`; commands e queries têm handlers específicos; Application não
conhece EF/ASP.NET; Domain permanece puro; Infrastructure implementa as portas externas; UseCases e
query services antigos foram removidos; e todos os testes permanecem aprovados.

## Situação do MainDDD após a PR 103

- todos os controllers usam Commands ou Queries por meio de `ISender`;
- não existem tipos `UseCase` na Application;
- Application não referencia Entity Framework, ASP.NET ou Infrastructure;
- o serviço de comandos de Documento permanece como orquestrador interno do contexto, atrás dos
  handlers, por reunir a transação indivisível de documento, estoque e Outbox;
- contratos HTTP e migrations permanecem compatíveis;
- testes de arquitetura impedem a reintrodução das dependências removidas.
