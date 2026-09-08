# Roadmap — Agentes, autenticação e controle de acesso

## Objetivo

Evoluir a autenticação provisória para um modelo dinâmico no qual um agente recebe capacidades por papéis e pode ser especializado por permissões individuais, sem acoplar a autorização às tabelas físicas do banco ou ao Microsoft Entra ID.

## Princípios

- Um agente representa usuário humano, conta de serviço, aplicação ou integração.
- A identidade usada para entrar é independente das permissões do agente.
- O primeiro nível de autorização é herdado dos papéis de acesso.
- O segundo nível permite concessões e negações específicas por agente.
- Recursos e ações formam um catálogo extensível, sem enum fixo de CRUD.
- Ausência de permissão significa acesso negado.
- A API é a autoridade de segurança; o frontend apenas reflete a autorização.
- Cada etapa deve manter a aplicação executável e possuir testes e migration próprios quando necessário.

## Resolução de uma permissão

1. Negação individual explícita.
2. Permissão individual explícita.
3. Permissão herdada dos papéis.
4. Negação por padrão.

## Sequência planejada

### PR 41 — Agentes de acesso e identidades

**Estado:** planejada  
**Dependências:** nenhuma

**Escopo**

- Criar `AgenteAcesso`.
- Criar `IdentidadeAcesso`.
- Suportar identidades Development, Local, Microsoft Entra ID e provedores futuros.
- Tornar a identidade externa opcional.
- Preservar o usuário DEV durante a transição.
- Separar estado do agente, identidade e método de autenticação.

**Critérios de aceite**

- Um agente pode existir sem Entra ID.
- Um agente pode possuir mais de uma identidade.
- A remoção de uma identidade não remove permissões nem histórico do agente.
- Migration e testes de domínio concluídos.

### PR 42 — Catálogo dinâmico de recursos e ações

**Estado:** planejada  
**Dependências:** PR 41

**Escopo**

- Criar `RecursoSistema` com código estável, módulo, tela e rota.
- Criar `AcaoSistema` para ações como ler, criar, alterar, excluir, processar e importar.
- Permitir hierarquia módulo → tela → funcionalidade.
- Cadastrar inicialmente os recursos existentes.
- Preparar registro automático por futuras entidades e formulários low-code.

**Critérios de aceite**

- Recursos e ações podem ser adicionados sem alterar enums.
- Códigos utilizados pela API são únicos e estáveis.
- Recursos inativos deixam de ser concedidos sem perder histórico.

### PR 43 — Papéis e permissões herdadas

**Estado:** planejada  
**Dependências:** PR 42

**Escopo**

- Criar `PapelAcesso`, `PapelPermissao` e `AgentePapel`.
- Criar papéis iniciais: Proprietário, Administrador, Gestor, Operador e Consulta.
- Converter o mapa de permissões atual para dados persistidos.
- Adicionar concorrência otimista às configurações.

**Critérios de aceite**

- Um agente pode possuir mais de um papel.
- Permissões dos papéis são combinadas.
- O proprietário inicial mantém acesso administrativo completo.

### PR 44 — Especialização por agente

**Estado:** planejada  
**Dependências:** PR 43

**Escopo**

- Criar `AgentePermissao`.
- Suportar estados não configurada, permitida e negada.
- Permitir capacidades adicionais e restrições individuais.
- Preservar a origem da decisão para apresentação e auditoria.

**Critérios de aceite**

- Uma negação individual prevalece sobre permissões herdadas.
- Uma permissão individual complementa os papéis.
- A remoção da especialização restaura o comportamento herdado.

### PR 45 — Serviço central de autorização

**Estado:** planejada  
**Dependências:** PR 44

**Escopo**

- Calcular permissões efetivas.
- Informar se a decisão é herdada, individual ou negada.
- Adicionar cache por agente e invalidação após alterações.
- Impedir a remoção do último agente com administração completa.

**Critérios de aceite**

- Todas as combinações de precedência possuem testes.
- Alterações invalidam o cache imediatamente.
- O serviço não depende do frontend ou de um provedor de identidade.

### PR 46 — Matriz visual de permissões

**Estado:** planejada  
**Dependências:** PR 45

**Escopo**

- Listar agentes à esquerda.
- Exibir papéis associados e permissões efetivas à direita.
- Agrupar recursos por módulo e tela.
- Mostrar estados herdada, permitida, negada e não configurada.
- Usar a coluna `Todos` apenas como comando para marcar as ações visíveis.
- Permitir pesquisa, expansão dos grupos e salvamento com concorrência otimista.

**Critérios de aceite**

- A matriz é construída pelo catálogo de recursos e ações.
- Nenhuma coluna de ação é fixa no componente.
- O usuário consegue distinguir permissões herdadas das individuais.

### PR 47 — Aplicação das permissões na API

**Estado:** planejada  
**Dependências:** PR 45 e PR 46

**Escopo**

- Criar policies dinâmicas por recurso e ação.
- Proteger os endpoints existentes.
- Cobrir ações especiais: processar, cancelar, importar, configurar e auditar.
- Registrar tentativas negadas relevantes.
- Garantir bootstrap administrativo antes de ativar o bloqueio.

**Critérios de aceite**

- Chamadas sem capacidade retornam `403 Forbidden`.
- O frontend não é necessário para impedir acesso indevido.
- Proprietário e administrador não são bloqueados durante a migração.

### PR 48 — Aplicação das permissões no frontend

**Estado:** planejada  
**Dependências:** PR 47

**Escopo**

- Ocultar módulos e menus não autorizados.
- Proteger rotas.
- Ocultar ou desabilitar ações como Novo, Editar, Excluir e Processar.
- Reutilizar o serviço de permissões efetivas.

**Critérios de aceite**

- A interface não oferece ações que a API recusará.
- A tentativa direta de navegar para uma rota protegida é tratada.
- Atualizações administrativas refletem sem exigir novo deploy.

### PR 49 — Autenticação local

**Estado:** planejada  
**Dependências:** PR 41 e PR 47

**Escopo**

- Associar credencial local ao agente.
- Armazenar apenas hash seguro da senha.
- Implementar bloqueio por tentativas, ativação, inativação e troca de senha.
- Substituir o login fictício fora do fluxo exclusivo de Development.

**Critérios de aceite**

- Senhas nunca são armazenadas ou registradas em texto puro.
- O bloqueio não altera papéis ou permissões.
- A autenticação entrega à autorização o identificador do agente.

### PR 50 — Preparação para Microsoft Entra ID

**Estado:** planejada  
**Dependências:** PR 49

**Escopo**

- Criar contrato para provedores externos.
- Preparar configuração do Microsoft Entra ID sem exigir uma conta Azure agora.
- Permitir associação posterior entre uma identidade Entra e um agente existente.
- Documentar ativação e migração da identidade local.

**Critérios de aceite**

- Associar o Entra ID não cria outro agente automaticamente quando já existe vínculo confirmado.
- Papéis, especializações e histórico permanecem no agente.
- O provedor externo pode permanecer desativado em DEV.

## Fora deste roadmap

- Administração de tenants ou infraestrutura dedicada por cliente.
- Provisionamento de Microsoft Entra ID ou recursos Azure.
- Políticas condicionais avançadas por valor de campo ou linha de registro.
- Delegação temporária de acesso e fluxos formais de aprovação.

Essas capacidades poderão ser acrescentadas depois sobre o catálogo de recursos, ações e agentes sem alterar o fundamento deste roadmap.
