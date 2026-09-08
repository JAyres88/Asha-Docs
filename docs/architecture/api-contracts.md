# Contratos protegidos da API

## Objetivo

A API é a fronteira pública do MainDDD. A modularização interna não deve alterar rotas, formatos JSON, códigos HTTP ou regras que já tenham consumidores no frontend e, futuramente, no MainIntegration.

Os testes de contrato hospedam a API em memória. Eles validam a fronteira HTTP sem depender do SQL Server local.

## O que constitui um contrato

Um contrato inclui mais do que o DTO de uma resposta:

- rota, verbo HTTP e parâmetros de rota ou consulta;
- campos publicados, tipos, paginação e nomes JSON;
- códigos de resposta e formato de erro;
- cabeçalhos obrigatórios, como `Idempotency-Key`;
- política de autorização aplicada ao endpoint;
- comportamento para recurso inexistente, conflito e validação inválida.

## Contratos atualmente protegidos

- Consultas de categorias e documentos preservam o envelope de paginação.
- Tipos de documento preservam os campos consumidos pelo cliente.
- Recursos inexistentes retornam `404 Not Found`.
- A criação de documento sem `Idempotency-Key` retorna `400 Bad Request`.
- Repetição da mesma chave com conteúdo diferente retorna `409 Conflict`.
- Usuário de desenvolvimento inválido retorna `401 Unauthorized`.
- Endpoints administrativos e operacionais aplicam as permissões exigidas por suas políticas.

## Evolução segura

1. Acrescente campos opcionais antes de tornar um campo obrigatório.
2. Não altere o significado de um campo já publicado.
3. Para mudanças incompatíveis, publique uma nova versão do contrato.
4. Adicione ou ajuste o teste de contrato antes da refatoração interna.
5. Valide o contrato no pipeline com testes unitários, de contrato e de composição modular.

## Execução

```powershell
dotnet test Tests/MainAPI.ContractTests/MainAPI.ContractTests.csproj
```

Os testes de contrato complementam testes de domínio e de aplicação: eles verificam o que um consumidor realmente observa pela HTTP.
