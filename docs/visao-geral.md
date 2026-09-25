# Visão geral dos módulos

## Propósito

A plataforma Gen combina uma base de gestão com produtos especializados. O cliente escolhe os produtos que utilizará, enquanto os cadastros corporativos necessários à operação permanecem no Gen.2 - Gestão. Cada cliente deve poder receber uma instalação própria, com configuração e versão independentes das demais.

| Produto | Papel na plataforma | Responsabilidade principal |
| --- | --- | --- |
| Gen.1 - Portal | Entrada comercial | Apresentar os módulos, receber os dados iniciais da organização e acompanhar a preparação do ambiente. |
| Gen.2 - Gestão | Base operacional | Manter cadastros compartilhados, documentos, estoque, administração e o contexto global de acesso aos módulos. |
| Gen.3 - Ponto de Venda | Operação de loja | Executar vendas, caixa e consultas dos cadastros necessários à venda; enviar os fatos da operação para integração. |
| Gen.4 - Identity | Identidade | Autenticar pessoas e serviços, emitir tokens e manter uma sessão de acesso entre os aplicativos da instalação. |
| Gen.5 - Integração | Mediação entre produtos | Receber, rotear e acompanhar eventos e contratos de sincronização entre Gestão, PDV e futuros módulos. |
| Gen.6 - Onboarding & Deploy | Preparação técnica | Receber a solicitação do Portal, registrar seu estado e preparar ou associar a instalação do cliente. |

## Como os produtos se relacionam

O Portal inicia a jornada de aquisição. O cliente informa a organização, os responsáveis e os produtos desejados. O Gen.6 recebe o pedido de preparação e devolve o andamento e os endereços de acesso. O Gen.4 autentica o administrador e os usuários convidados. O Gen.2 mantém os cadastros básicos e o contexto global de módulos e direitos. O Gen.3 consulta os cadastros que lhe foram disponibilizados e envia eventos de venda pelo Gen.5.

O PDV depende dos cadastros comuns da Gestão, como locais, tipos de saldo, pessoas e produtos, mas não deve editar essas entidades. A escrita pertence ao sistema responsável pelo cadastro. A integração distribui somente os dados habilitados e transforma os eventos operacionais em documentos ou atualizações no destino. O mesmo padrão deve orientar produtos futuros.

## Estrutura interna do Gen.2

O Gen.2 é um monólito modular: uma API e publicação reúnem bibliotecas separadas para Catálogo, Pessoas, Precificação, Documentos, Estoque, IdentidadeAcesso e Administração. Os módulos têm contratos e camadas próprias; a persistência SQL Server é compartilhada dentro dessa instalação para preservar transações de negócio. Isso não significa compartilhar o banco entre clientes.

## Acesso e direitos

O Gen.4 fornece a identidade usada pelos aplicativos de uma instalação. Após o login, a Gestão determina o contexto da organização e os módulos liberados; cada aplicativo aplica suas permissões específicas tanto na interface quanto nas APIs. A demonstração utiliza uma conta de consulta, sem poderes de cadastro ou administração.

A criação de uma organização e a entrada em uma organização existente são caminhos distintos. O fluxo comercial completo de pagamento, convite e ativação automática ainda depende de implementação e integração adicionais.

## Produtos futuros

Projetos, CRM, Serviços e MRP II são propostas de módulos especializados. Eles podem ter aplicações e versões próprias, mas deverão usar os cadastros comuns da Gestão e os contratos do Gen.5. Sua presença na vitrine não significa que estejam prontos para contratação ou implantação.

## Repositórios

- [Gen.1 - Portal](https://github.com/JAyres88/Gen.1-Portal)
- [Gen.2 - Gestão](https://github.com/JAyres88/Gen.2-Gestao)
- [Gen.3 - Ponto de Venda](https://github.com/JAyres88/Gen.3-Ponto-de-Venda)
- [Gen.4 - Identity](https://github.com/JAyres88/Gen.4-Identity)
- [Gen.5 - Integração](https://github.com/JAyres88/Gen.5-Integracao)
- [Gen.6 - Onboarding & Deploy](https://github.com/JAyres88/Gen.6-Onboarding-Deploy)

Veja também a [visão de infraestrutura](visao-infraestrutura.md).
