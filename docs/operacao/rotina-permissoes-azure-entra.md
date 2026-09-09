# Rotina de manutenção de permissões Azure e Microsoft Entra

Este documento define as rotinas recorrentes de acesso da plataforma Main. Identificadores de assinatura, grupos de recursos, cofres, contas administrativas e nomes de identidades são mantidos apenas no inventário operacional privado.

## Princípios

- Cada pessoa usa sua própria identidade Microsoft; credenciais não são compartilhadas.
- Uma permissão é concedida no menor escopo possível: recurso, grupo de recursos e somente então assinatura.
- Deploy usa identidade gerenciada e federação do GitHub; não usa senha de usuário.
- Segredos de aplicativos ficam somente no Azure Key Vault.
- Toda mudança registra responsável, motivo, escopo e data de revisão.

## Toda semana

1. Revisar atividades administrativas e alertas de orçamento, custo previsto e falhas de deploy.
2. Conferir atribuições `Owner` e `Contributor` fora do grupo de recursos previsto.
3. Confirmar que workflows GitHub usam somente a identidade federada e branch autorizada.
4. Investigar tentativas de login e mudanças de acesso sem responsável identificado.

## Todo mês

1. Revisar atribuições de função Azure e administradores do Microsoft Entra.
2. Conferir consentimentos entre aplicativos e remover os que não são utilizados.
3. Validar redirecionamentos de login de cada ambiente.
4. Conferir versões e validade de segredos no Key Vault.
5. Confirmar que logs, configurações, artefatos e repositórios não contêm segredos.
6. Revisar o escopo da identidade de deploy; ela deve permanecer restrita ao ambiente correspondente.

## A cada trimestre

1. Confirmar que cada acesso humano ainda é necessário.
2. Remover acessos temporários, colaboradores e fornecedores sem atividade atual.
3. Revisar registros de aplicativo, proprietários e permissões.
4. Validar a recuperação de acesso administrativo com uma segunda conta de contingência.
5. Revisar alertas de custo e os destinatários responsáveis.

## Rotação anual e por incidente

Credenciais de servidor devem ser rotacionadas ao menos 30 dias antes do vencimento:

1. Criar uma nova credencial no registro do aplicativo.
2. Gravar uma nova versão do segredo no Key Vault.
3. Atualizar o ambiente para consumir a nova versão.
4. Validar login e chamada autenticada.
5. Revogar a credencial anterior após a validação.

Em caso de suspeita de vazamento, revogue a credencial imediatamente, gere outra, atualize o Key Vault, reinicie a aplicação afetada e revise os logs de auditoria.

## Inclusão e remoção de pessoas

Para conceder acesso, identificar a pessoa pelo e-mail corporativo, definir a função mínima e registrar a data de revisão. Para remover, revogar funções Azure e grupos Microsoft Entra, retirar a pessoa dos proprietários de aplicativos quando aplicável e revogar sessões em situações de risco.

## Novo módulo

Antes do primeiro deploy de um módulo:

1. Criar seu registro de aplicativo no Microsoft Entra.
2. Definir escopos próprios e permissões delegadas mínimas.
3. Cadastrar redirecionamentos específicos por ambiente.
4. Guardar credenciais no Key Vault.
5. Restringir a identidade de deploy ao grupo de recursos do ambiente.
6. Registrar e revisar as permissões antes do deploy.

O inventário com nomes de recursos e comandos operacionais fica fora do repositório público. Nunca publique valores de segredos, tokens ou identificadores administrativos.
