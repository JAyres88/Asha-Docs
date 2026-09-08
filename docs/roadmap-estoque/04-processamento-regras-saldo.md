# Processamento das Regras de Saldo

## Objetivo

Aplicar todos os lançamentos planejados pelo documento de forma atômica, idempotente e protegida contra concorrência.

## Escopo

- Processar operações `Somar` e `Subtrair` na ordem configurada.
- Confirmar todas as linhas em uma única transação.
- Impedir consumo concorrente acima do saldo permitido.
- Não duplicar lançamentos quando o processamento for repetido.
- Estornar criando lançamentos inversos sem apagar o histórico.
- Migrar configurações antigas de entrada, saída e transferência para regras equivalentes.
- Remover o contrato antigo somente depois da compatibilidade estar comprovada.

## Critérios de aceite

- Falha em qualquer lançamento desfaz toda a operação.
- Repetir o mesmo comando não altera novamente os saldos.
- Estorno recompõe todos os saldos atingidos.
- Saldos informativos são movimentados, mas não alteram totais físicos ou disponíveis.
- Transferências continuam preservando a quantidade física total.
- Testes cobrem múltiplas somas e subtrações, falha parcial, concorrência e estorno.

## Dependência

Implementar depois de **Fotografia das Movimentações do Documento**.
