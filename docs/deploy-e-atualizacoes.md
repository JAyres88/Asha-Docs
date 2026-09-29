# Deploy e atualização da instalação Asha — AS IS

Este guia descreve a instalação que roda neste computador. O GitHub armazena o código e coordena a publicação; o Docker Desktop executa os serviços aqui; o Cloudflare Tunnel publica os endereços web. Acesso pela Internet não significa que as aplicações estejam hospedadas no GitHub ou no Cloudflare. O computador, o Docker Desktop e a sessão do usuário precisam estar disponíveis.

## Onde fica cada parte

| Local | Conteúdo e função |
| --- | --- |
| `C:\Asha\Asha-Portal`, `Asha-Gestao`, `Asha-Ponto-de-Venda`, `Asha-Identity`, `Asha-Integracao` | Cópias de trabalho dos aplicativos. Portal, Gestão, PDV e Integração acompanham `DEV`; Identity acompanha `main` nesta instalação. |
| `C:\Asha\Asha-Onboarding-Deploy` | API e worker de onboarding, manifesto de versões, testes e esteira central de publicação. |
| `C:\Asha\asha_deployExec` | Scripts que instalam e configuram o executor oficial do GitHub. |
| `C:\Asha\Asha-Docs` | Documentação. |
| `C:\Asha\ActionsRunner` e `C:\Asha\Tools` | Executor `asha-local` e ferramentas usadas pelos trabalhos de publicação. Ele inicia ao entrar na sessão do Windows. |
| `C:\Asha\Deployment` | Configuração e manifesto da instalação, arquivos privados, histórico de releases e registro do último sucesso. Não é parte dos repositórios. |

O endereço antigo em `%LOCALAPPDATA%\Asha` é uma ligação de compatibilidade para `C:\Asha`; os arquivos físicos estão em `C:\Asha`. Credenciais, segredos e configurações locais não devem ser enviados ao GitHub. SQL Server e RabbitMQ guardam seus dados em volumes Docker; mover as pastas de código não move nem reinicia esses volumes.

## Do desenvolvimento à publicação

1. A mudança é feita no repositório do aplicativo, passa por PR e pelos testes de sua CI. Um merge ou um `git pull` atualiza código-fonte, **não** os contêineres.
2. Em `Asha-Onboarding-Deploy/config/release.json`, uma PR define o SHA completo aprovado de cada aplicativo, os Dockerfiles e os serviços que receberão as imagens. Essa seleção fixa a versão; a esteira não publica automaticamente a ponta móvel de uma branch.
3. A CI do Onboarding & Deploy valida o manifesto e os scripts em um executor do GitHub. Para publicar, um responsável inicia manualmente **Actions → Validar e publicar instalação Asha → Run workflow**, escolhe `component` (`all` ou um aplicativo) e deixa `plan_only=false`.
4. O trabalho de deploy é recebido pelo runner `asha-local` em `C:\Asha\ActionsRunner`. Ele clona, em uma pasta isolada, os SHAs definidos no manifesto. As cópias de trabalho em `C:\Asha\Asha-*` não são alteradas pelo deploy.
5. A esteira verifica as revisões instaladas e as migrations, compila e testa o código selecionado, constrói imagens Docker e as envia ao registry local. Em seguida atualiza somente os serviços escolhidos no Swarm, fixando cada imagem por digest.
6. Por fim, aguarda os serviços, verifica os endereços HTTPS e a autenticação e testa a demonstração Portal → Gestão → PDV. Só após essas verificações registra o release como bem-sucedido em `C:\Asha\Deployment\releases\<id>` e atualiza o manifesto instalado.

Uma migration nova bloqueia a publicação até que backup e aplicação da mudança no banco sejam tratados e o SHA seja aprovado em `migrationsApproved.<aplicativo>` no `deployment.json`. O deploy não aplica migrations automaticamente. Em caso de falha após alterar serviços, o script tenta restaurar as imagens anteriores; isso não reverte dados do banco.

## Atualização local

Para obter o código atual no computador, use `git pull --ff-only` na branch de trabalho do repositório em `C:\Asha`. Confira a branch antes: Portal, Gestão, PDV e Integração usam `DEV`; Identity, Onboarding & Deploy, Docs e deployExec usam `main` nesta instalação. Exemplo:

```powershell
git -C C:\Asha\Asha-Gestao branch --show-current
git -C C:\Asha\Asha-Gestao pull --ff-only
```

Isso serve para editar e inspecionar o código. Para atualizar o serviço que roda localmente, promova o SHA em `release.json` e execute a publicação central. Primeiro é possível conferir o plano sem trocar imagens:

```powershell
gh workflow run local-swarm.yml --repo JAyres88/Asha-Onboarding-Deploy --ref main -f component=gestao -f plan_only=true
```

Depois de revisar o plano, repita com `plan_only=false` ou use o botão do GitHub Actions. O runner precisa aparecer **online** em `Asha-Onboarding-Deploy → Settings → Actions → Runners`. A conta local, o Docker Desktop e o Swarm precisam estar ativos. O runner usa uma conexão de saída com o GitHub; nenhuma porta de entrada do Docker ou do runner deve ser aberta para a web.

## Atualização vista de fora do computador

O GitHub Actions é o **controle remoto da publicação**, não um segundo ambiente de hospedagem. Quando o runner local termina, os serviços atualizados continuam neste PC. Traefik encaminha os hostnames aos serviços internos e o Cloudflare Tunnel conecta os endereços públicos ao gateway por saída da rede local. A publicação de código não exige refazer DNS nem as rotas do túnel enquanto hostnames e serviços não mudarem.

Os acessos preparados são `https://portal.sysgen.win`, `https://gestao.sysgen.win`, `https://pdv.sysgen.win`, `https://login.sysgen.win` e `https://integration.sysgen.win`. SQL Server, RabbitMQ, registry e APIs internas permanecem sem rota pública própria. Se o PC ou o túnel parar, os endereços externos deixam de responder mesmo que a última execução do GitHub tenha sido bem-sucedida.

## Conferência e recuperação

```powershell
docker service ls
docker service ps main_mainddd
docker service inspect main_mainddd --format '{{.Spec.TaskTemplate.ContainerSpec.Image}}'
gh run list --repo JAyres88/Asha-Onboarding-Deploy --workflow local-swarm.yml --limit 5
```

Confirme `1/1` para os serviços, a imagem/digest esperada, o resultado da Action e o funcionamento dos endereços e do login. `C:\Asha\Deployment\last-successful-release` identifica a última publicação concluída; a pasta `releases` guarda os planos e resultados. Se uma publicação falhar, verifique a etapa da Action e o estado real do serviço antes de tentar de novo. Um rollback de imagem não substitui backup ou plano de recuperação do banco.

O [Onboarding & Deploy](https://github.com/JAyres88/Asha-Onboarding-Deploy/blob/main/docs/continuous-delivery.md) documenta as etapas técnicas e os limites da esteira; o [deployExec](https://github.com/JAyres88/asha_deployExec) documenta a instalação e a recuperação do runner.
