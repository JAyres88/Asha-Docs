# Roteiro de configuração e publicação no Azure

Este roteiro prepara o ambiente de teste da plataforma Main para receber atualizações automáticas de Billing, MainDDD e PDV. O MainIntegration participa como serviço interno, sem endereço público.

## Resultado esperado

Ao final, a assinatura Azure terá um grupo de recursos de teste com os seguintes destinos:

| Produto | Destinos publicados | Função |
| --- | --- | --- |
| MainBilling | Portal e API | aquisição, licenças e entrada comercial |
| MainDDD | Portal e API | cadastros compartilhados e seleção de módulos |
| MainPDV | Portal e API | operação de ponto de venda |
| MainIntegration | API e worker | troca de dados entre produtos |

Cada portal terá uma URL própria. Cada API ficará isolada e será chamada pelo seu portal ou por serviços autorizados.

## 1. Confirmar a conta e o limite de custo

1. Entrar na assinatura Azure usada para testes.
2. Confirmar o grupo de recursos `rg-mainsyst-teste`, na região Brazil South.
3. Manter o orçamento `orcamento-mainsyst-teste` ativo e ligado ao grupo de ações de custo.
4. Configurar alertas em 50%, 70% e 85% do crédito disponível.
5. No alerta de 85%, pausar as atualizações automáticas e reduzir as aplicações de teste para zero réplicas.

O orçamento avisa sobre o consumo; a pausa das aplicações precisa ser uma ação de automação associada ao alerta. Ela deve ser testada antes de hospedar clientes.

## 2. Criar os recursos de hospedagem

No mesmo grupo de recursos, criar:

1. Um Azure Container Registry para armazenar as imagens versionadas.
2. Um ambiente Azure Container Apps para executar as aplicações.
3. Uma aplicação de contêiner para cada portal e API pública.
4. Uma aplicação de contêiner sem entrada pública para o worker do MainIntegration.
5. Uma conta de banco Azure SQL para a base compartilhada da plataforma.
6. Um banco inicial de teste e uma política de backup compatível com o ambiente de demonstração.

As aplicações de teste devem iniciar com mínimo de zero e máximo de uma réplica. Isso permite parar o consumo de processamento quando ninguém estiver usando o ambiente.

## 3. Configurar dados e segredos

1. Guardar a cadeia de conexão do banco apenas no Key Vault `kvmainstest11b95`.
2. Dar às aplicações apenas permissão de leitura do segredo de que precisam.
3. Configurar as variáveis de ambiente das APIs com a cadeia de conexão e os dados do Entra ID.
4. Configurar cada portal com a URL da sua API.
5. Nunca registrar senhas, cadeias de conexão ou segredos nos repositórios ou nas variáveis visíveis do GitHub.

A base compartilhada deve ser usada apenas pelos serviços já preparados para o contrato comum. Enquanto o PDV ainda depender do seu esquema local, ele deve apontar para uma base de teste própria. A unificação definitiva depende do plano de integração entre MainDDD e MainPDV.

## 4. Configurar Entra ID

1. Confirmar os registros de aplicação `MainDDD API` e `MainPDV`.
2. Criar ou ajustar o registro de aplicação da API do Billing.
3. Cadastrar as URLs públicas de retorno de cada portal.
4. Autorizar os escopos entre portal e API necessários para cada produto.
5. Manter a conta administradora principal e a conta de demonstração conforme a rotina de permissões.
6. Validar o acesso ao MainDDD após a entrada pelo Billing e a disponibilidade do PDV para uma licença ativa.

Consulte também [Rotina de permissões Azure e Entra ID](rotina-permissoes-azure-entra.md).

## 5. Preparar a atualização automática

Cada repositório deve ter:

1. Arquivo de contêiner para cada aplicação executável.
2. Pipeline acionado por merge na branch de entrega.
3. Identidade federada do GitHub com permissão de publicar imagens e atualizar somente o grupo de recursos de teste.
4. Variáveis de repositório para assinatura, tenant, registro de imagens e nomes das aplicações.
5. Publicação da imagem com o identificador do commit, para permitir retorno a uma versão anterior.

O MainDDD já possui a preparação inicial no pipeline de publicação Azure. Billing, PDV e MainIntegration devem seguir o mesmo padrão, adequando seus projetos de entrada.

## 6. Executar a primeira publicação

A primeira publicação deve seguir esta ordem:

1. Criar o banco e aplicar as migrações do MainDDD.
2. Publicar a API e o portal do MainDDD.
3. Validar login, seleção de módulos e a tela de Contas de acesso.
4. Publicar Billing e validar a entrada para o MainDDD.
5. Preparar a base de teste do PDV e publicar sua API e portal.
6. Validar o acesso ao PDV a partir da licença no MainDDD.
7. Publicar a API e o worker do MainIntegration.
8. Executar uma sincronização de teste e registrar o resultado.

## 7. Validação de entrega

Antes de divulgar os endereços de teste, confirmar:

- Cada URL pública abre usando HTTPS.
- Cada portal chama apenas sua própria API configurada.
- APIs não expõem Swagger ou diagnósticos de desenvolvimento publicamente.
- Segredos estão apenas no Key Vault.
- A conta administradora acessa as configurações e a conta de demonstração acessa somente os módulos previstos.
- O orçamento envia alertas para o canal definido.
- Uma atualização por merge é publicada e pode ser revertida para a imagem anterior.

## Operação recorrente

Depois da primeira implantação, o fluxo normal será: abrir PR, validar, fazer merge na branch de entrega e acompanhar o pipeline. Mudanças de banco, permissões Entra ID e segredos devem seguir este roteiro e a rotina de permissões antes da publicação.
