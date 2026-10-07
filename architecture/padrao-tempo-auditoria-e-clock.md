# Padrão de timestamps, auditoria e relógio das aplicações

## Objetivo

Definir uma base coerente para registrar quando os dados e fatos de negócio foram criados ou alterados e para executar automações com uma referência de tempo testável. O padrão se aplica aos serviços Asha que persistem dados: Portal, Gestão, Ponto de Venda, Identity, Integração e Onboarding & Deploy, respeitando o domínio e o ciclo de atualização de cada aplicação.

Este documento especifica a direção de implementação. A inclusão dos campos e dos agendadores nas aplicações requer PRs de código e migrações próprias; este documento não afirma que essas mudanças já foram implantadas.

## Decisões

1. Instantes são persistidos como UTC, com offset explícito na aplicação (DateTimeOffset).
2. O tempo corrente é obtido por uma dependência injetada baseada em TimeProvider do .NET, em vez de chamadas dispersas a DateTime.Now, DateTime.UtcNow ou DateTime.Today.
3. Entidades persistidas recebem metadados de criação e atualização quando fizer sentido para o ciclo de vida delas.
4. O instante de uma operação de negócio (OccurredAtUtc) é distinto do instante em que o registro foi gravado (CreatedAtUtc).
5. Cada aplicação é dona das automações do seu domínio. A Integração transporta contratos/eventos entre aplicações, mas não se torna o relógio central nem passa a executar regras internas de outros módulos.
6. Agendamentos relevantes têm estado persistido, execução idempotente e proteção contra execução concorrente.

## Modelo de auditoria

Entidades mutáveis que representam dados de negócio devem ter:

- CreatedAtUtc: definido uma vez na inclusão e imutável.
- UpdatedAtUtc: atualizado em cada alteração persistida.
- CreatedByUserId e UpdatedByUserId: identidade estável do autor, quando a alteração vier de um usuário autenticado.
- OccurredAtUtc: usado em registros que representam um fato de negócio com instante próprio, como venda, movimentação, fechamento, recebimento ou evento de integração.

Os campos de autoria podem ficar vazios para processos automáticos ou origens externas. Nesses casos, registrar uma identidade técnica padronizada e rastreável quando o mecanismo atual permitir; nunca atribuir a operação a um usuário humano por conveniência.

Campos de criação/atualização não substituem o histórico transacional. Movimentos de estoque, eventos publicados/consumidos e operações financeiras preservam seus próprios fatos, identificadores e instantes. Não sobrescrever o instante de negócio original ao reprocessar uma mensagem.

Não adicionar timestamps cegamente a valores imutáveis, tabelas de junção puramente técnicas, projeções descartáveis ou dados externos que a aplicação apenas espelha. Se um registro puder ser alterado ou for necessário para auditoria, aplicar o padrão. Cada exceção deve ser justificada no modelo do domínio.

## Relógio da aplicação

Registrar TimeProvider no contêiner de dependências e usá-lo nos casos de uso, handlers, serviços de domínio que dependam do tempo, interceptadores de persistência e serviços em segundo plano. Exemplo:

    var now = timeProvider.GetUtcNow();

O registro padrão usa TimeProvider.System. Testes podem fornecer um relógio controlável, como FakeTimeProvider do pacote Microsoft.Extensions.TimeProvider.Testing, para validar expiração, vencimentos, limites de período e disparo de rotinas sem esperas reais.

Entidades não devem consultar diretamente o relógio do sistema. A data/hora atual também não deve ser calculada na interface para decisões de negócio. Para comparações de datas civis, converter o instante para o fuso configurado para a organização ou unidade e aplicar uma regra explícita de calendário.

O relógio técnico responde “que instante é agora?”. Calendário operacional responde “a que dia, período ou fechamento esse instante pertence?”. Regimes fiscais, horário de corte e calendário por unidade pertencem a configurações/regras do domínio; não devem ser codificados como fuso local da máquina. Manter os servidores e contêineres em UTC.

## Persistência e EF Core

Aplicações com EF Core devem preencher os metadados em um ponto uniforme do pipeline de persistência, por exemplo por meio de um SaveChangesInterceptor ou mecanismo já existente. O mecanismo deve:

- usar o TimeProvider injetado;
- preencher CreatedAtUtc apenas na inclusão;
- manter CreatedAtUtc imutável em atualizações;
- preencher UpdatedAtUtc em toda inclusão e alteração;
- preencher autoria a partir de um serviço de usuário atual, sem acoplar o domínio a claims HTTP;
- não substituir OccurredAtUtc de um fato já registrado.

No PostgreSQL, mapear instantes para timestamp with time zone (o tipo timestamptz). Enviar e ler valores como UTC. Revisar explicitamente os mapeamentos e os provedores de cada serviço; não depender de conversões implícitas do fuso do host.

## Eventos e mensagens

Contratos publicados devem transportar messageId, occurredAtUtc, producer e a versão do contrato. O consumidor preserva o instante de ocorrência recebido e registra também o momento de recebimento/processamento quando necessário. Retentativas mantêm o mesmo identificador do evento original e não geram uma nova ocorrência de negócio.

A outbox deve registrar timestamps de criação e publicação; a inbox, timestamps de recebimento e processamento. Esses instantes servem ao diagnóstico e à latência, sem substituir o timestamp do fato de negócio.

## Automações e fechamentos

Cada módulo mantém as rotinas próprias do seu negócio. Por exemplo, Gestão é responsável pelos seus fechamentos de gestão; o PDV, por rotinas de operação do caixa. A Integração pode distribuir o resultado como evento quando houver consumidor, sem transferir a propriedade da rotina.

Uma rotina que precise sobreviver a reinícios deve ter definição e estado de execução persistidos, incluindo próximo instante, última execução e resultado. Cada execução precisa de uma chave idempotente (por exemplo, rotina + unidade + período) e de uma forma de impedir que duas instâncias processem a mesma ocorrência ao mesmo tempo. Em uma instalação com uma única réplica, também se deve preservar essa proteção para permitir escala futura.

Um BackgroundService simples pode servir para tarefas internas de baixa criticidade, desde que a execução não dependa de memória volátil. Se houver calendários configuráveis, recorrências, recuperação de execuções perdidas ou múltiplas réplicas, escolher um agendador persistente após verificar as dependências existentes. A escolha de biblioteca deve ser uma decisão transversal antes de cada módulo adotar uma solução diferente.

## Migração dos dados existentes

Cada aplicação cria sua própria migração compatível com o seu banco. A migração deve preservar registros e seus valores atuais, preencher metadados para os dados legados com uma regra explícita e documentada e permitir implantação sem perda do histórico.

Não presumir que um timestamp legado sem fuso represente UTC. Antes do backfill, identificar a convenção usada na aplicação e no banco. Se não for possível reconstruir o instante com segurança, preservar o valor legado e marcar a origem/precisão conforme o modelo permitir, em vez de inventar um horário. Implantar de forma compatível: adicionar colunas, preencher dados, atualizar gravações/leitura e só depois tornar campos obrigatórios quando isso for seguro.

## Plano de implementação por aplicação

1. Gestão: campos comuns nas entidades mutáveis e timestamps de fatos para documentos, movimentos de saldo e fechamentos; migrar e validar histórico sem alterar quantidades ou documentos.
2. Ponto de Venda: timestamps de venda, itens, pagamentos, cancelamentos e devoluções, correlacionados ao instante de negócio recebido ou gerado pelo PDV.
3. Integração: aplicar o mesmo padrão a outbox/inbox, mensagens, rotas e execuções de processamento, preservando occurredAtUtc do producer.
4. Identity: auditar criação/alteração de usuários, concessões, convites, autenticação multifator e eventos de sessão sem registrar segredos.
5. Portal: auditar cadastro de leads/pedidos e alterações de estado, mantendo o instante do envio do formulário.
6. Onboarding & Deploy: registrar solicitação, início, término e resultado de provisionamentos/deploys; não gravar tokens ou credenciais nos metadados.
7. Agendadores: implementar rotinas em cada aplicação que realmente tenha automação de negócio. Não criar um processo de agendamento em serviços que ainda não possuam rotinas justificadas.

A sequência deve ser executada por PRs independentes por aplicação, coordenadas por uma definição comum e implantadas com migrações compatíveis. Alterações de contrato de evento devem ser adicionadas de forma retrocompatível durante a transição.

## Critérios de aceite

- Não há decisões de negócio dependentes de chamadas diretas ao relógio do sistema fora do provedor de tempo.
- Instantes são persistidos e transmitidos em UTC, e exibidos segundo o fuso configurado.
- Criação é imutável; atualização é atualizada uniformemente; timestamp do fato de negócio é preservado.
- A autoria usa identidade autenticada ou técnica rastreável e não inventa usuário.
- Testes controlam o tempo sem esperar o relógio real.
- Migrações preservam os dados e declaram como tratar timestamps legados sem fuso.
- Rotinas sobrevivem a reinício quando necessário, podem ser repetidas com segurança e não executam duas vezes para a mesma chave/período.
- Cada automação tem um módulo proprietário explícito.