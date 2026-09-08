# Tipos de saldo como classificadores de estoque

## Objetivo

`TipoSaldo` classifica a posição de um produto em estoque. Ele não representa entrada, saída ou neutralidade.

`FinalidadeDocumento.Gerencial` descreve a finalidade de valor zero de um documento e permanece independente de seu efeito sobre o estoque.

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

As operações de estoque disponíveis são:

1. `NaoMovimenta`;
2. `Entrada`;
3. `Saida`;
4. `Transferencia`.

Os valores persistidos do enum permanecem compatíveis com o histórico já gravado.

### Finalidade do documento

Um documento pode ter finalidade gerencial e, independentemente disso, movimentar ou não movimentar estoque.

## Transferência classificatória

Uma mudança de classificação produz dois lançamentos dentro da mesma transação:

```text
Saída:   Disponível            10 unidades
Entrada: Reservado para venda  10 unidades
```

O total físico permanece igual, mas a composição dos saldos muda. Caso qualquer lançamento falhe, nenhum deles deve ser confirmado.

## Regras vigentes

- Documento configurado como `NaoMovimenta` não gera movimento de estoque.
- Transferência gera uma saída e uma entrada correlacionadas, de forma atômica e idempotente.
- Cancelamento ou estorno cria movimentos inversos sem apagar o histórico.
- Concorrência otimista protege a posição de estoque contra consumo simultâneo indevido.

Esta estrutura é a base para documentos, reservas, separação e integrações com canais externos.
