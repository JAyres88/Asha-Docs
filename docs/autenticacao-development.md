# Autenticação multitenant em desenvolvimento

Enquanto o Microsoft Entra External ID não está configurado, a Main usa um esquema local exclusivo do ambiente `Development`.

## Usuários mock

| Usuário | Papel |
| --- | --- |
| `proprietario@main.local` | Proprietário |
| `operador@main.local` | Operador |
| `consulta@main.local` | Consulta |

Os usuários podem ser consultados em `GET /api/dev/autenticacao/usuarios`.

## Conta de demonstração Microsoft Entra

A demonstração pública usa a conta Microsoft Entra `dev@mainsystoutlook.onmicrosoft.com`. Ela não possui funções ou permissões administrativas na Azure e deve ser associada somente ao tenant de demonstração quando a autenticação federada estiver habilitada no código.

As credenciais e o atalho de acesso são apresentados na experiência de demonstração do Billing. Essa conta não substitui os usuários mock locais deste documento.

O usuário `proprietario@main.local` é assumido automaticamente quando o cabeçalho não é informado. Esse valor fica exclusivamente no `appsettings.Development.json`.

## Swagger

1. Execute os endpoints normalmente para usar o Proprietário padrão.
2. Para testar outro papel, clique em **Authorize**.
3. Informe o email do Operador ou Consulta no campo `X-Dev-User`.

O endpoint `GET /api/dev/autenticacao/contexto` mostra o usuário, papel e tenant reconhecidos na requisição.

## Segurança

- O esquema local só é registrado em `Development`.
- Nenhuma senha é armazenada ou simulada.
- Usuários inativos e tenants inativos são rejeitados.
- Fora de `Development`, a aplicação falha ao iniciar enquanto o Microsoft Entra não estiver configurado.

## Migração futura para Microsoft Entra

Controllers e casos de uso dependem de `IUsuarioContext`, `ITenantContext` e políticas da aplicação. A integração futura substituirá apenas o esquema de autenticação local por `Microsoft.Identity.Web`, preservando os contratos internos e as permissões.

