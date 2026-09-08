# Tipos de saldo como classificadores de estoque

## Objetivo

Eliminar a ambiguidade do termo `Neutra` e consolidar `TipoSaldo` como classificação configurável da posição do produto.

No mesmo vocabulário, a finalidade de valor zero passa a se chamar `Gerencial`: ela descreve o propósito do documento e permanece independente de seu efeito sobre o estoque.

## Conceitos

### Tipo de saldo

Classifica onde ou para qual finalidade uma quantidade está controlada, por exemplo:

- Disponível;
- Reservado para venda;
- Em separação;
- Bloqueado;
- Em inspeção;
- Em trânsito;
- Expedição.

O tipo de saldo não possui natureza de entrada, saída ou neutralidade.

### Movimento de estoque

Cada lançamento individual possui natureza `Entrada` ou `Saída` e referencia produto, embalagem, local e tipo de saldo.

### Operação do tipo de documento

Substituir o vocabulário `Neutra` por `NaoMovimenta`:

1. `NaoMovimenta`;
2. `Entrada`;
3. `Saida`;
4. `Transferencia`.

Os valores persistidos do enum devem permanecer compatíveis. A alteração inicial é semântica e não deve reinterpretar registros existentes.

### Finalidade do documento

Substituir `FinalidadeDocumento.Neutra` por `Gerencial`, preservando o valor numérico zero. Assim, um documento pode ter finalidade gerencial e, independentemente disso, movimentar ou não movimentar estoque.

## Transferência classificatória

Uma mudança de classificação produz dois lançamentos dentro da mesma transação:

```text
Saída:   Disponível            10 unidades
Entrada: Reservado para venda  10 unidades
```

O total físico permanece igual, mas a composição dos saldos muda. Caso qualquer lançamento falhe, nenhum deles deve ser confirmado.

## Escopo

- Renomear `OperacaoEstoqueDocumento.Neutra` para `NaoMovimenta`, preservando seu valor numérico.
- Renomear `FinalidadeDocumento.Neutra` para `Gerencial`, preservando seu valor numérico.
- Remover referências visuais e textuais a saldo ou operação “neutra”.
- Garantir que `TipoSaldo` permaneça apenas como classificador configurável.
- Formalizar a transferência entre locais e/ou tipos de saldo como saída e entrada atômicas.
- Atualizar DTOs, validações, Swagger, frontend, seeds e documentação.
- Revisar processamento, cancelamento e estorno de documentos.
- Adicionar testes de domínio, serviço e contrato.

## Fora do escopo

- Criar tipos fixos de saldo no código.
- Presumir que todo saldo de um local esteja disponível para venda.
- Implementar reservas automáticas dos canais integrados ou workflow de separação.
- Reescrever migrations já aplicadas.

## Critérios de aceite

- Não existe a opção “Neutra” nas telas ou contratos publicados.
- Documento configurado como `NaoMovimenta` não cria movimento de estoque.
- Transferência gera exatamente uma saída e uma entrada correlacionadas.
- Transferência é atômica e idempotente.
- Soma física é preservada quando origem e destino pertencem ao mesmo local.
- Concorrência otimista impede consumo simultâneo indevido.
- Cancelamento ou estorno cria movimentos inversos, sem apagar o histórico.
- Banco existente continua compatível e o EF Core não apresenta mudança pendente não planejada.

## Dependências e ordem

Esta estrutura permanece no ERP como base para documentos, reservas, separação e integrações com canais externos.
