# Idempotência na criação de documentos

## Objetivo

`POST /api/documentos` exige o cabeçalho `Idempotency-Key`. A chave impede que duplo clique, timeout ou repetição de rede criem dois documentos para a mesma intenção de negócio.

```http
POST /api/documentos HTTP/1.1
Idempotency-Key: 8ba81e90-5247-4cd6-9b7c-bc338c55dc13
Content-Type: application/json
```

O cliente gera uma chave ao iniciar a criação e reutiliza a mesma chave em todas as tentativas daquela operação. Uma nova criação recebe uma nova chave.

## Comportamento

| Situação | Resultado |
| --- | --- |
| Chave nova | Cria o documento. |
| Mesma chave e mesmo conteúdo | Devolve o documento já criado, sem outra gravação. |
| Mesma chave e conteúdo diferente | Retorna `409 Conflict`. |
| Chave ausente, vazia ou com mais de 100 caracteres | Retorna `400 Bad Request`. |

A API compara uma assinatura da requisição para distinguir uma repetição legítima de uma reutilização incorreta da chave.

## Garantia de persistência

`Documentos.IdempotencyKey` possui índice único filtrado: `IX_Documentos_IdempotencyKey`. A proteção no banco cobre tentativas concorrentes que cheguem antes da primeira resposta HTTP.

A idempotência é complementar à concorrência otimista: a primeira evita duas criações da mesma intenção; a segunda protege alterações posteriores de um documento já existente.

## Uso em integrações

O mesmo princípio será usado no contrato `VendaPdvConcluidaV1`. A referência da venda do PDV e a chave de idempotência permitirão que o MainDDD crie ou encontre uma única **Venda Cliente Final**, mesmo que o MainIntegration reentregue uma mensagem.
