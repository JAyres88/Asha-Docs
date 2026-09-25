# Visão de infraestrutura

## Modelo de hospedagem

O primeiro provedor de hospedagem é um computador local com Docker Desktop e Docker Swarm. Um gateway Traefik encaminha as requisições aos serviços. O domínio sysgen.win usa Cloudflare DNS e um Cloudflare Tunnel para levar HTTPS ao gateway sem expor SQL Server, RabbitMQ ou APIs internas diretamente à internet.

No ambiente atual, o Portal, o gateway, o túnel e o registro local de imagens formam a base de publicação. A stack de demonstração contém Gen.2, Gen.3, Gen.4, Gen.5 e Gen.6, SQL Server e RabbitMQ. O SQL Server guarda bancos separados por aplicação dentro da instalação; RabbitMQ transporta mensagens entre serviços. Docker secrets e configurações fora do Git fornecem senhas e parâmetros de execução.

| Serviço | Função |
| --- | --- |
| Docker Desktop + Swarm | Executar e administrar serviços, redes, volumes, configurações e segredos. |
| Registro local de imagens | Guardar imagens versionadas antes da implantação. |
| Traefik | Selecionar o serviço pela rota de entrada e encaminhar o tráfego. |
| Cloudflare Tunnel | Conectar os endereços públicos ao gateway por uma conexão de saída. |
| Gen.1 - Portal | Vitrine e entrada do registro da organização. |
| Gen.6 API + worker | Persistir pedidos, executar etapas de preparação e publicar seu estado. |
| Gen.4 - Identity | Login OIDC/OAuth2 e tokens para usuários e serviços. |
| Gen.2 - Gestão | Cadastros e operação corporativa. |
| Gen.3 - Ponto de Venda | API e interface de vendas. |
| Gen.5 - Integração | API, interface operacional e distribuição de eventos. |
| SQL Server | Persistência de cada aplicação. |
| RabbitMQ | Mensageria interna. |

Hoje os endereços públicos preparados são portal.sysgen.win, gestao.sysgen.win, pdv.sysgen.win e login.sysgen.win. Integration, Provisioning, SQL Server e RabbitMQ permanecem internos. A existência do DNS e do túnel não comprova que uma versão específica esteja em execução; a implantação e a saúde dos serviços devem ser verificadas separadamente.

## Alocação quando um cliente se registra: desenho alvo

A intenção é criar uma instalação completa por cliente, com versão e customizações independentes. O computador pode hospedar várias instalações, mas compartilhar o equipamento não deve misturar seus dados, segredos, rede interna nem ciclo de atualização. O Portal e o Gen.6 continuam como serviços de controle do provedor.

1. **Registro:** o Portal recebe a organização, o administrador inicial, os módulos escolhidos e preferências de operação. Envia ao Gen.6 um pedido com identificador único. A confirmação comercial e as condições de contratação devem ser verificadas antes de alocar recursos.
2. **Reserva:** o Gen.6 verifica a capacidade disponível do host, reserva CPU, memória e armazenamento estimados e cria um identificador interno estável da instalação. Pedidos repetidos com o mesmo identificador não podem criar uma segunda instalação.
3. **Preparação:** o Gen.6 gera a definição de stack do cliente a partir de um modelo versionado. O modelo escolhe as imagens aprovadas para aquele cliente e seus módulos licenciados. Uma versão customizada pode ser atribuída só a essa stack.
4. **Isolamento:** o Swarm cria a stack, rede interna, volumes persistentes, segredos e bancos do cliente. O desenho exige uma instância SQL Server e um volume de dados próprios por cliente; RabbitMQ e Identity também pertencem à instalação do cliente. O gateway, o túnel e o registro de imagens podem ser compartilhados pelo provedor.
5. **Inicialização:** migrations e dados iniciais são aplicados aos bancos do cliente. O Gen.4 cadastra o administrador inicial; a Gestão registra a organização e a licença, e os módulos contratados recebem suas configurações.
6. **Publicação:** o Gen.6 cria rotas e endereços próprios do cliente no gateway e no túnel. Os nomes públicos devem ser únicos e os retornos de login devem apontar para a instalação correta. Nenhuma API interna ou banco recebe rota pública.
7. **Verificação:** o Gen.6 aguarda saúde dos serviços, login, acesso à Gestão e aos módulos licenciados. Só então marca o pedido como concluído, devolve os links ao Portal e dispara a comunicação ao administrador.
8. **Falha e repetição:** cada etapa grava estado e diagnóstico. Uma falha permite repetir a etapa sem duplicar volumes, usuários, rotas ou cobranças. Recursos reservados para uma tentativa cancelada precisam ser liberados explicitamente.

A instalação de cada cliente pode ser atualizada ou revertida de forma independente. O provisionador deve manter o mapeamento entre organização, identificador da stack, versão das imagens, volumes, segredos, rotas e licença. A seleção de host e os limites de capacidade devem impedir que um novo cliente comprometa as instalações já ativas.

## O que está implementado agora

O Gen.6 já possui API, worker, armazenamento durável de solicitações, estados de execução, tentativas e retorno de endereços. O Portal envia os dados iniciais. A stack Docker da demonstração é gerada e implantada por scripts operacionais; as imagens são construídas e guardadas no registro local. O acesso público tem gateway e Cloudflare Tunnel configurados.

No modo local atual, a etapa LocalInstallationReadyStep devolve endereços de uma instalação que já existe. Ela não executa as etapas 2 a 7 acima: não reserva capacidade, não cria uma stack nova por organização, não separa banco e mensageria por cliente, não cria rotas próprias e não cadastra automaticamente o administrador. Portanto, o registro de um novo cliente ainda não resulta em alocação dinâmica independente. Essa é a principal implementação pendente no Gen.6.

O código também conserva um adaptador antigo de provisionamento externo, desabilitado na stack Docker atual. Ele não deve ser interpretado como a solução de alocação local descrita aqui.

## Limites operacionais

Em um único computador, as instalações continuam dependendo do mesmo hardware, energia e conexão. Instâncias SQL Server, aplicações e mensageria próprias por cliente consomem recursos mesmo sem tráfego. O Gen.6 precisa aplicar limites de CPU, memória, disco e número de instalações, medir o uso real e recusar novas alocações quando não houver capacidade segura. Backups e restauração devem tratar cada cliente separadamente.

A [visão geral](visao-geral.md) explica as responsabilidades de cada produto.
