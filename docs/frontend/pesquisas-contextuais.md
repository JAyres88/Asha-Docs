# Pesquisas contextuais por lupa e modal

## Objetivo

Padronizar toda seleção de registros relacionados no frontend com um campo de pesquisa, botão de lupa e modal contextual reutilizável, seguindo a experiência já aplicada a Tipo de Documento e Pessoa em Documentos.

## Regra de interface

- Campos que referenciam outra entidade usam `LookupField` e `LookupDialog`.
- Combos permanecem apenas para valores pequenos e fechados, como status, finalidade, ordenação e outros enums.
- O identificador técnico não deve ser digitado ou exibido como única informação para o usuário.
- O valor selecionado mostra um resumo legível e pode ser limpo quando a regra permitir.

## Componente agnóstico

Evoluir os componentes existentes para suportar:

- pesquisa no servidor durante a digitação;
- paginação e ordenação;
- colunas e texto de resumo definidos pelo contexto;
- seleção simples e, quando necessário, múltipla;
- estados de carregamento, vazio e erro;
- navegação por teclado e foco acessível;
- confirmação por duplo clique ou botão Selecionar;
- ação opcional para abrir o cadastro completo do registro;
- cancelamento sem modificar o valor anterior.

O componente controla a experiência, mas cada página fornece consulta, colunas, regras de elegibilidade e retorno selecionado.

## Aplicações mapeadas

### Documentos e movimentações

- Tipo de documento;
- Pessoa;
- Endereço de entrega cadastrado;
- Local de origem e destino;
- Tipo de saldo de origem e destino, filtrado pelo local;
- Tabela de preço;
- Produto e embalagem/apresentação;
- troca da tabela de preço de um item já inserido.

### Tabela de preço

- Produto;
- Embalagem/apresentação vinculada ao produto;
- seleção de itens que receberão preço e validade.

### Produto e categoria

- Embalagem no grid de apresentações do produto;
- Produto no grid de produtos da categoria;
- Categoria quando houver seleção relacional no cadastro de produto.

### Locais e estoque

- Tipo de saldo associado ao local;
- filtros de produto, local e tipo de saldo na consulta de estoque.

### Tipos de documento

- Local padrão de origem e destino;
- Tipo de saldo padrão de origem e destino.

### Implementações futuras

Qualquer novo parâmetro relacional deve usar o mesmo padrão, incluindo usuário, perfil, estabelecimento, condição de pagamento e documento relacionado.

## Regras contextuais

- Embalagens exibidas devem pertencer ao produto selecionado.
- Preços devem respeitar tabela, vigência, produto e embalagem.
- Tipos de saldo devem estar habilitados no local selecionado.
- Registros inativos não aparecem por padrão, com opção explícita de consulta quando pertinente.
- A autorização do usuário também restringe resultados e ações de abertura/edição.
- A API continua validando o identificador recebido; o modal não substitui validação de domínio.

## Fora do escopo

- Substituir combos de status e enums por modal.
- Editar registros completos dentro do modal.
- Carregar catálogos inteiros antecipadamente no navegador.
- Criar consultas genéricas que permitam acessar entidades sem autorização.

## Critérios de aceite

- Nenhum campo relacional mapeado permanece como combo simples ou textbox de ID.
- Todas as pesquisas possuem lupa visível e modal consistente.
- Digitação refina os dados consultando a API com debounce e cancelamento da requisição anterior.
- Grandes volumes são paginados no servidor.
- Seleção e limpeza atualizam corretamente identificador e descrição.
- Campos dependentes são limpos quando o registro pai muda.
- Modal funciona por mouse e teclado e devolve o foco ao campo de origem.
- Testes de componentes cobrem abertura, pesquisa, seleção, cancelamento e dependências.
- Fluxos de Documento, Tabela de Preço, Produto, Categoria, Local, Tipo de Documento e Estoque são verificados.

## Ordem recomendada

1. Evoluir o componente genérico e seu contrato.
2. Adequar Documentos e seus itens.
3. Adequar Tabelas de Preço, Produto e Categoria.
4. Adequar Local, Tipo de Documento e Estoque.
5. Remover seletores relacionais antigos e executar regressão visual.
