# Separação entre MainDDD e MainPDV

O módulo operacional de ponto de venda foi removido do MainDDD e transferido
para o produto independente MainPDV.

## Responsabilidade do MainDDD

- pessoas e clientes;
- produtos, categorias, embalagens e códigos de barras;
- tabelas de preço;
- documentos empresariais;
- locais, tipos de saldo e estoque;
- recebimento e conciliação futura das operações originadas no MainPDV.

## Responsabilidade do MainPDV

- configuração do terminal e do caixa;
- abertura e fechamento de sessão;
- venda de balcão;
- pagamentos e periféricos;
- operação offline e sincronização;
- integração fiscal de NFC-e;
- configuração específica do local utilizado pelo caixa.

## Histórico de banco

As migrations que criaram a fundação do PDV permanecem no histórico para que
um banco novo possa reproduzir toda a evolução do schema. A migration de
remoção elimina as tabelas e colunas operacionais ao final da atualização.

Contratos explícitos de sincronização serão introduzidos posteriormente; o
MainDDD não deve referenciar agregados internos do MainPDV.
