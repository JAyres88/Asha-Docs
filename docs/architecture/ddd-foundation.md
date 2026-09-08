# Fundação DDD

O MainDDD aplica estes fundamentos em seus módulos. Eles orientam a separação entre domínio, aplicação, infraestrutura e contratos públicos.

## Domínio

- `Entity<TId>` define identidade e igualdade por tipo e identificador.
- `AggregateRoot<TId>` controla a coleta de eventos de domínio.
- `IDomainEvent` representa um fato ocorrido no domínio.
- `DomainError` identifica uma regra por código e mensagem.
- `DomainException` comunica uma violação de regra para a fronteira HTTP.
- `Guard` centraliza verificações básicas usadas na construção do modelo.

## Aplicação

- `ICommand<TResult>` e `ICommandHandler` representam operações que alteram estado.
- `IQuery<TResult>` e `IQueryHandler` representam consultas sem alteração de estado.
- `IUnitOfWork` representa a confirmação de uma unidade de persistência.
- `IClock` remove a dependência direta de `DateTime.UtcNow` dos casos de uso.

## Infraestrutura

- `AppDbContext` implementa `IUnitOfWork` sem alterar o modelo do Entity Framework.
- `SystemClock` é a implementação padrão de `IClock`.
- `ExceptionHandlingMiddleware` converte `DomainException` em HTTP 422 e mantém o formato de erro já consumido pelos clientes.

## Regras de adoção

1. Controllers continuam responsáveis apenas pela fronteira HTTP.
2. Casos de uso coordenam agregados e dependências externas.
3. Agregados protegem seus invariantes e publicam eventos.
4. Repositórios são definidos por abstrações e implementados na infraestrutura.
5. DTOs da API não são entidades de domínio.
6. Alterações em contratos públicos são protegidas por testes de contrato.
