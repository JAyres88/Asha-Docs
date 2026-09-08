# Produto, embalagem e apresentações compostas

## Decisão

Substituir gradualmente a associação identificada apenas por `ProdutoId + EmbalagemId` por uma entidade com identidade própria chamada `ProdutoApresentacao`.

`Embalagem` continuará sendo o cadastro genérico do formato físico, como pacote, caixa, garrafa, saco, lata ou palete. Peso, conteúdo, código de barras e conversão pertencem à apresentação específica do produto.

## Motivação

O mesmo produto pode usar mais de uma apresentação com o mesmo tipo de embalagem:

- Biscoito — pacote de 200 g;
- Biscoito — pacote de 500 g;
- Biscoito — caixa com 24 pacotes de 200 g.

A chave atual `ProdutoId + EmbalagemId` não permite dois pacotes diferentes para o mesmo produto.

Também existem apresentações mistas, nas quais uma caixa contém apresentações diferentes. Nesse caso, um único fator de conversão não descreve adequadamente a composição.

## Modelo proposto

### ProdutoApresentacao

- `Id`;
- `ProdutoId`;
- `EmbalagemId`;
- `Nome` ou nome alternativo;
- `FatorConversao` para apresentações simples;
- `PesoLiquidoKg`;
- `PesoBrutoKg`;
- `CodigoBarras`;
- `EmbalagemPadrao`;
- `PermiteVenda`;
- `PermiteCompra`;
- `Status`;
- controle de concorrência.

### ProdutoApresentacaoComponente

- `ApresentacaoPaiId`;
- `ApresentacaoComponenteId`;
- `Quantidade`.

Essa entidade descreve a composição de caixas, kits, fardos e paletes.

## Exemplo de caixa mista

Para uma caixa com 5 pacotes de 500 g e 10 pacotes de 200 g:

| Apresentação pai | Componente | Quantidade |
|---|---|---:|
| Caixa mista | Pacote 500 g | 5 |
| Caixa mista | Pacote 200 g | 10 |

O peso líquido calculado da caixa será `5 × 0,5 kg + 10 × 0,2 kg = 4,5 kg`.

A quantidade de componentes não deve ser deduzida de um único fator. A composição será a fonte oficial para apresentações compostas.

## Regras de domínio

- Uma apresentação simples utiliza fator de conversão e não possui componentes.
- Uma apresentação composta possui ao menos um componente.
- A mesma apresentação componente não pode aparecer repetida na mesma composição.
- Uma apresentação não pode conter a si própria, direta ou indiretamente.
- A composição não pode formar ciclos.
- Quantidades dos componentes devem ser maiores que zero.
- Código de barras deve ser único quando informado.
- Apenas apresentações ativas e permitidas para a operação podem ser selecionadas em documentos.
- Peso líquido de uma composição pode ser calculado pelos componentes; divergências com peso declarado devem ser validadas.
- Documento deve gravar uma fotografia do nome, conversão e composição relevante no momento da emissão.

## Impactos

### Preços

`ProdutoPreco` passará a referenciar `ProdutoApresentacaoId`, permitindo preço distinto para pacote de 200 g, pacote de 500 g e caixa mista.

### Documentos

`DocumentoItem` passará a referenciar a apresentação selecionada. A quantidade informada representa a quantidade da apresentação vendida ou comprada.

### Estoque

O saldo deve definir se é controlado pela apresentação ou pela unidade-base. A conversão e a explosão da composição devem ocorrer em um serviço de domínio único, evitando cálculos diferentes entre documentos e estoque.

### Frontend

O cadastro de Produto terá um grid de apresentações. O editor permitirá:

- selecionar uma embalagem cadastrada;
- criar várias apresentações do mesmo tipo;
- informar peso, conversão e código de barras;
- marcar a apresentação como simples ou composta;
- montar uma composição pesquisando outras apresentações em modal contextual.

## Estratégia de migração

1. Criar `ProdutoApresentacao` com chave própria e migrar cada `ProdutoEmbalagem` atual para uma apresentação.
2. Criar `ProdutoApresentacaoComponente`.
3. Alterar preços para referenciar a apresentação.
4. Alterar documentos e seus itens para referenciar a apresentação e preservar a fotografia comercial.
5. Adequar saldos e movimentos de estoque.
6. Atualizar APIs, DTOs, validações, mocks e frontend.
7. Remover as chaves e estruturas antigas somente após a migração dos dados e validação dos contratos.

## Critérios de aceite

- O mesmo produto aceita duas apresentações com a mesma embalagem, como pacote de 200 g e pacote de 500 g.
- É possível cadastrar uma caixa simples ou mista.
- A composição mista aceita quantidades diferentes de cada apresentação componente.
- Ciclos e autorreferências são rejeitados.
- Tabela de preço diferencia cada apresentação.
- Documento seleciona e registra a apresentação correta.
- Movimentação de estoque utiliza uma única política de conversão/composição.
- Dados existentes são migrados sem perda.
