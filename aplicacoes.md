# Aplicações

Responsabilidades das aplicações Asha conforme o código verificado em 5 de outubro de 2026. A disponibilidade dos fluxos depende da versão e da configuração implantadas.

## Asha Portal

Entrada comercial da plataforma. Apresenta produtos, recebe dados da organização e do contato, registra módulos de interesse e oferece caminhos para demonstração e acesso aos aplicativos.

O fluxo comercial atual envia pedidos de interesse à Integração, que os encaminha à Gestão para registro e tratamento pelas regras de documentos. O formulário não processa pagamentos. O projeto também mantém contratos e telas de acompanhamento de onboarding, dependentes da habilitação do serviço de preparação.

Cadastros operacionais de produtos, pessoas, estoque e vendas pertencem às aplicações de negócio.

## Asha Gestão

Base administrativa e operacional compartilhada. Mantém os cadastros e as regras consumidos pelos produtos e o contexto global de acesso aos módulos.

| Módulo | Responsabilidade |
| --- | --- |
| Catálogo | Produtos, apresentações e cadastros relacionados. |
| Pessoas | Pessoas e vínculos usados nas operações. |
| Precificação | Estruturas e regras de preços. |
| Documentos | Tipos de documento, registros e processamento de operações. |
| Estoque | Tipos de saldo, movimentações e seus efeitos. |
| IdentidadeAcesso | Conta, organização, papéis, módulos e direitos. |
| Administração | Configurações e funções administrativas. |

Recebe os pedidos comerciais encaminhados pelo Portal e pela Integração, com processamento e baixa conforme a configuração de documentos. Disponibiliza cadastros e recebe operações relacionadas à sincronização com o PDV.

## Asha Ponto de Venda

Atende à operação de loja: seleção de produtos, montagem e processamento de vendas, pagamentos, caixa e rotinas operacionais. Organiza também documentos, estoque, pós-venda, configuração e integração.

Consulta dados compartilhados necessários à operação e publica eventos para a Integração. Os cadastros corporativos de referência pertencem à Gestão; dados e regras específicos da venda pertencem ao PDV.

O projeto contém uma fronteira de integração fiscal, mas isso não implica disponibilidade de todos os provedores ou modos de emissão. A preferência por operação temporária sem internet coletada no Portal também não comprova um fluxo completo de sincronização offline.

## Asha Identity

Provedor local de identidade. Gerencia usuários, autenticação, sessão, papéis, clientes de serviço e tokens. Reúne serviços de convite e comunicação por e-mail, dependentes de configuração SMTP.

O administrador de identidade pode conceder ou revogar o papel técnico da Integração, com auditoria. Administrar o Identity não concede automaticamente privilégios na Gestão ou na Integração. A conta de demonstração tem finalidade de consulta e não recebe esse acesso técnico.

## Asha Integração

Media contratos entre aplicações. Recebe eventos e pedidos, encaminha operações, acompanha entregas e fornece administração para diagnóstico, reconciliação e tratamento de falhas.

Participa da sincronização entre Gestão e PDV, da entrega de eventos operacionais e do encaminhamento de pedidos comerciais do Portal. Seu banco registra dados técnicos de recepção e entrega; cadastros e regras de negócio permanecem nas aplicações responsáveis.

A interface administrativa exige usuário autorizado, escopo administrativo e IntegrationAdministrator. Credenciais de serviço para eventos não liberam administração técnica.

## Asha Onboarding & Deploy

Reúne duas responsabilidades:

- **Preparação técnica:** a API recebe solicitações e o worker executa etapas, acompanha estados e tentativas e pode associar a organização à instalação configurada.
- **Implantação:** scripts e workflows selecionam revisões, constroem imagens, atualizam serviços e verificam o resultado.

Quando habilitado, o fluxo local pode preparar o acesso na Gestão e devolver endereços da instalação. O manifesto básico mantém esse processamento desabilitado. Há abstrações para provisionamento externo; a criação automática de uma instalação dedicada por cliente não está concluída.

## Executor de implantação

O asha_deployExec é um projeto operacional. Prepara o host e instala/configura o GitHub Actions Runner oficial. A operação acontece no GitHub Actions; não há agente remoto próprio nem aplicação de negócio para o usuário final.

## Fluxos entre aplicações

| Fluxo | Participação |
| --- | --- |
| Pedido comercial | Portal coleta interesse → Integração encaminha → Gestão registra e processa. |
| Acesso aos produtos | Identity autentica → Gestão fornece contexto global → aplicativo verifica a operação. |
| Venda | PDV executa → Integração recebe e encaminha eventos → Gestão aplica contratos e efeitos previstos. |
| Preparação habilitada | Portal solicita → Onboarding executa etapas → Gestão recebe a preparação de acesso → Portal apresenta andamento e endereços. |
| Atualização | Onboarding & Deploy seleciona revisões → runner executa → Swarm atualiza serviços. |

Projetos, CRM, Serviços e MRP II aparecem como propostas na vitrine. Não são aplicações implantáveis na stack descrita.

## Repositórios

Veja também [Infraestrutura](infraestrutura.md) e [Visão Técnica](visao-tecnica.md).
