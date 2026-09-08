# Idempotência na criação de documentos

O endpoint `POST /api/documentos` exige o cabeçalho `Idempotency-Key`. Ele impede que
uma repetição da mesma requisição, causada por duplo clique, timeout ou nova
tentativa de rede, crie documentos duplicados.

```http
POST /api/documentos HTTP/1.1
Idempotency-Key: 8ba81e90-5247-4cd6-9b7c-bc338c55dc13
Content-Type: application/json
```

O frontend deve gerar uma chave única ao iniciar uma nova tentativa de criação e
reutilizar essa mesma chave em todas as repetições daquela tentativa. Ao começar
outro documento, deve gerar uma nova chave.

## Comportamento

- Chave nova: o documento é criado normalmente.
- Mesma chave e mesmo conteúdo: o documento já criado é devolvido, sem nova gravação.
- Mesma chave e conteúdo diferente: a API devolve `409 Conflict`.
- Chave ausente, vazia ou maior que 100 caracteres: a API devolve `400 Bad Request`.

A restrição única `IX_Pedidos_IdempotencyKey` garante a proteção mesmo quando
duas requisições iguais chegam ao servidor ao mesmo tempo. Documentos anteriores à
migration permanecem válidos, com a chave nula.
