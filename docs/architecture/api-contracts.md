# Contratos protegidos da API

Os testes de contrato hospedam a API em memória e validam a fronteira HTTP sem acessar o SQL Server local.

## Contratos iniciais

- `GET /api/categorias` mantém o envelope de paginação.
- `GET /api/documentos` mantém o envelope de paginação.
- `GET /api/tipos-documento` mantém os campos publicados para o frontend.
- Recursos inexistentes continuam retornando `404 Not Found`.
- A emissão de documento sem `Idempotency-Key` retorna `400 Bad Request`.
- Um usuário de desenvolvimento inválido recebe `401 Unauthorized`.

## Execução

```powershell
dotnet test Tests/MainAPI.ContractTests/MainAPI.ContractTests.csproj
```

Novos contratos relevantes devem receber um teste antes de qualquer refatoração que atravesse a fronteira HTTP.
