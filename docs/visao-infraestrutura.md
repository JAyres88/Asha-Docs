# Visão de infraestrutura — AS IS

Este documento registra a configuração local e o comportamento que podem ser verificados no código e no manifesto de implantação. Ele não define a arquitetura futura de alocação por cliente.

## Hospedagem e publicação

O computador local utiliza Docker Desktop e Docker Swarm. O Traefik encaminha as requisições para as aplicações pela rota de entrada. O domínio sysgen.win usa Cloudflare DNS e Cloudflare Tunnel para levar HTTPS ao gateway. SQL Server, RabbitMQ, Provisioning e APIs internas não têm rotas públicas dedicadas.

A configuração de implantação mantém um registro local de imagens e uma rede Docker chamada main-provider. O manifesto da stack main reúne os aplicativos, SQL Server e RabbitMQ. As imagens são construídas e versionadas por scripts do Asha Onboarding & Deploy. Docker secrets e arquivos locais fora do Git fornecem senhas e parâmetros de execução.

| Serviço | Uso na configuração local |
| --- | --- |
| Docker Desktop + Swarm | Executa serviços e administra redes, volumes, configurações e segredos. |
| Registro local de imagens | Armazena as imagens versionadas usadas pela stack. |
| Traefik | Encaminha o tráfego de entrada conforme o hostname. |
| Cloudflare Tunnel | Conecta os endereços públicos ao gateway por conexão de saída. |
| Asha Portal | Apresenta módulos e recebe o formulário da organização. |
| Asha Onboarding & Deploy API + worker | Persiste solicitações, executa etapas e retorna seu estado. |
| Asha Identity | Autentica usuários e serviços com OIDC/OAuth2. |
| Asha Gestão | Hospeda a API, a interface e os módulos da base operacional. |
| Asha Ponto de Venda | Hospeda a API e a interface de vendas. |
| Asha Integração | Hospeda a API, a interface e o processamento de integração. |
| SQL Server | Mantém bancos separados por aplicação nessa stack. |
| RabbitMQ | Transporta mensagens internas. |

Os hostnames preparados são portal.sysgen.win, gestao.sysgen.win, pdv.sysgen.win e login.sysgen.win. Todos chegam ao mesmo gateway pelo túnel. O gateway seleciona o serviço correspondente. DNS e túnel ativos não demonstram, por si só, que uma aplicação está saudável ou atualizada; a versão e a execução de cada serviço são verificadas no Swarm.

## Registro de uma organização no ambiente local

1. No Portal, a pessoa seleciona módulos e informa organização, contato comercial, administrador inicial, região de hospedagem e preferência de operação temporária sem internet.
2. O Portal envia ao Asha Onboarding & Deploy uma solicitação com identificador único. A API valida o pedido e o guarda no banco AshaProvisioning.
3. O worker busca a próxima solicitação pronta, grava o estado da etapa, registra tentativas e pode repetir uma etapa após uma falha.
4. Com LocalProvisioning habilitado, a etapa LocalInstallationReadyStep associa à solicitação os endereços da instalação local já configurada. Ela sempre inclui Gestão e acrescenta PDV ou Integração conforme os códigos de módulo informados.
5. O Asha Onboarding & Deploy conclui a operação e devolve estado e endereços ao Portal. A tela de ativação consulta esse estado e apresenta o acesso à Gestão quando disponível.

Esse é o processamento do registro presente no código. O planejamento de uma forma diferente de alocar recursos e separar instalações pertence à definição do TO BE.

## Persistência e operação

A stack local usa um SQL Server com bancos distintos para Identity, Gestão, PDV, Integração e Provisioning. O manifesto monta volume persistente para SQL Server e outro para RabbitMQ. Os serviços compartilham a rede interna main-provider, enquanto o gateway e o túnel atendem aos endereços externos.

O Swarm registra a escala desejada de cada serviço. Parar ou escalar uma aplicação para zero não remove imagens, volumes, bancos nem os serviços de infraestrutura. A saúde dos aplicativos, os logs e a versão das imagens devem ser verificados separadamente.

A [visão geral](visao-geral.md) descreve as responsabilidades dos produtos.
