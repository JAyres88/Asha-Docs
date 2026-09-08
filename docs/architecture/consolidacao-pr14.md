# Consolidação arquitetural — PR 14

## Objetivo

Encerrar o ciclo de migração incremental para DDD sem alterar os contratos HTTP já publicados nem reescrever migrations aplicadas.

## Estado consolidado

- **Domain** concentra entidades, agregados, objetos de valor, regras e eventos de domínio.
- **Application** concentra DTOs, validadores, casos de uso, comandos, consultas e abstrações.
- **Infrastructure** implementa persistência, consultas, relógio, eventos, outbox, auditoria e integrações técnicas.
- **WebAPI** é a fronteira HTTP e não contém regras de negócio.
- **Frontend** consome apenas contratos publicados pela API.

## Decisões preservadas

1. `Documento` permanece neutro; seus efeitos dependem do `TipoDocumento`.
2. Estoque somente é alterado por processamento documental explícito.
3. Escritas usam casos de uso e agregados; listagens podem usar serviços de consulta especializados.
4. `AppDbContext` implementa a unidade de trabalho e eventos são despachados na confirmação.
5. Outbox protege a publicação de efeitos assíncronos.
6. Idempotência e concorrência otimista continuam obrigatórias nos fluxos críticos.

## Limpezas realizadas

- Remoção do serviço legado de tipos de documento, substituído pelo caso de uso e serviço de consulta.
- Remoção de operações de repositório que permitiam alterar associações fora dos agregados.
- Remoção de mapeamentos manuais substituídos pelas entidades e DTOs consolidados.
- Padronização da composição de dependências em uma entrada fluente: `AddMainApplication`.
- Remoção de imports sem consumidor nos perfis de mapeamento.

## Compatibilidade e migrations

O histórico de migrations é intencionalmente preservado. Consolidar arquitetura não autoriza apagar ou reagrupar migrations potencialmente aplicadas em outros bancos. O snapshot deve permanecer compatível com o modelo atual e a verificação de mudanças pendentes faz parte da regressão desta PR.

## Regras para os próximos módulos

- Nova regra de negócio deve nascer no domínio ou em uma política de domínio.
- Orquestração pertence ao caso de uso, não ao controller.
- Dependência externa deve possuir contrato na camada interna e implementação na infraestrutura.
- Alteração de contrato HTTP exige teste de contrato.
- Alteração de persistência exige migration aditiva e teste proporcional ao risco.
- Não criar um padrão abstrato antes de existir uma variação real que o justifique.

## Verificação

```powershell
dotnet build Main.sln --no-restore
dotnet test Main.sln --no-build
dotnet ef migrations has-pending-model-changes --project src/MainAPI/MainAPI.csproj --startup-project src/MainAPI/MainAPI.csproj --no-build
```
