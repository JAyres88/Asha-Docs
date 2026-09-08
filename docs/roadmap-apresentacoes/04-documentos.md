# Etapa 4 — Apresentações nos documentos

## Dependências

Executar após as etapas 1 a 3.

## Escopo

- Alterar `DocumentoItem` para referenciar `ProdutoApresentacaoId`.
- Atualizar criação, edição, idempotência e processamento de documentos.
- Validar finalidade, status e permissões de venda ou compra da apresentação.
- Preservar no item uma fotografia do nome, embalagem, conversão, preço e composição relevante.
- Atualizar pesquisa contextual e troca de tabela de preço.
- Gerar no Pedido de Venda a necessidade prevista de materiais de embalagem.
- Associar volumes e packaging misto ao documento, sem criar composição cadastral.
- Manter essa necessidade exclusivamente no saldo de controle do documento.
- Não empenhar nem consumir estoque físico durante a emissão do pedido.

## Aceite

- Operador escolhe inequivocamente pacote de 200 g, pacote de 500 g ou caixa mista.
- Documento aberto pode ser editado mantendo a apresentação escolhida.
- Documento processado preserva os dados históricos mesmo após alteração cadastral.
- Repetição com a mesma chave idempotente não duplica itens nem documentos.
- O Pedido de Venda informa volumes e demanda prevista de packaging separadamente dos itens comerciais.
- Alteração ou cancelamento recalcula ou libera a previsão sem movimentação física.
