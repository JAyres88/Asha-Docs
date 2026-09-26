# Visão geral dos módulos

## Propósito

A plataforma Gen combina uma base de gestão com produtos especializados. O Portal apresenta os produtos e recebe os dados iniciais da organização. Os cadastros corporativos necessários à operação ficam no Gen.2 - Gestão.

| Produto | Papel na plataforma | Responsabilidade atual |
| --- | --- | --- |
| Gen.1 - Portal | Entrada comercial | Apresentar os módulos, receber dados da organização e mostrar o andamento da preparação. |
| Gen.2 - Gestão | Base operacional | Manter cadastros compartilhados, documentos, estoque, administração e contexto de acesso aos módulos. |
| Gen.3 - Ponto de Venda | Operação de loja | Executar vendas e caixa, consultar os cadastros necessários e enviar eventos da operação. |
| Gen.4 - Identity | Identidade | Autenticar pessoas e serviços e emitir tokens para os aplicativos. |
| Gen.5 - Integração | Mediação entre produtos | Receber, rotear e acompanhar eventos e contratos entre Gestão e PDV. |
| Gen.6 - Onboarding & Deploy | Preparação técnica | Registrar pedidos do Portal, processar suas etapas e associar a instalação local configurada. |

## Como os produtos se relacionam

O Portal recebe a organização, os responsáveis e os módulos selecionados e envia um pedido ao Gen.6. O Gen.6 registra o andamento e devolve os endereços da instalação configurada. O Gen.4 autentica usuários e serviços. O Gen.2 mantém os cadastros básicos e o contexto global de módulos e direitos. O Gen.3 consulta os cadastros disponibilizados pela Gestão e envia eventos de venda ao Gen.5.

O PDV consulta dados comuns da Gestão, como locais, tipos de saldo, pessoas e produtos. A edição desses cadastros pertence à Gestão. O Gen.5 media os contratos de sincronização e os eventos entre os produtos.

## Estrutura interna do Gen.2

O Gen.2 é um monólito modular: uma API e publicação reúnem bibliotecas separadas para Catálogo, Pessoas, Precificação, Documentos, Estoque, IdentidadeAcesso e Administração. Os módulos têm contratos e camadas próprias. Dentro da instalação local configurada, usam uma persistência SQL Server compartilhada para preservar transações de negócio.

## Acesso e direitos

O Gen.4 fornece a identidade usada pelos aplicativos da instalação. A Gestão fornece o contexto de módulos e direitos; os aplicativos aplicam as permissões em suas interfaces e APIs. A demonstração usa uma conta de consulta, sem poderes de cadastro ou administração.

A interface do Portal separa o cadastro de uma organização do acesso a uma organização existente. O formulário coleta os dados iniciais, mas não processa pagamentos.

## Produtos apresentados para desenvolvimento futuro

Projetos, CRM, Serviços e MRP II aparecem como propostas na vitrine. Ainda não são produtos implantáveis na stack local descrita aqui. Suas responsabilidades e sua integração serão especificadas quando cada produto for definido.

## Repositórios

- [Gen.1 - Portal](https://github.com/JAyres88/Gen.1-Portal)
- [Gen.2 - Gestão](https://github.com/JAyres88/Gen.2-Gestao)
- [Gen.3 - Ponto de Venda](https://github.com/JAyres88/Gen.3-Ponto-de-Venda)
- [Gen.4 - Identity](https://github.com/JAyres88/Gen.4-Identity)
- [Gen.5 - Integração](https://github.com/JAyres88/Gen.5-Integracao)
- [Gen.6 - Onboarding & Deploy](https://github.com/JAyres88/Gen.6-Onboarding-Deploy)

Veja também a [visão de infraestrutura](visao-infraestrutura.md).
