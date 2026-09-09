# Caminho feliz: implantação Azure pela CLI

Este roteiro cria um ambiente de teste privado para um produto Main com API, portal web e migrações. Execute os comandos no PowerShell autenticado com `az login`. Substitua os valores entre `<...>`; não inclua senhas ou cadeias de conexão em arquivos, commits ou variáveis do GitHub.

## 1. Variáveis da sessão

```powershell
$subscription = '<SUBSCRIPTION_ID>'
$location = 'brazilsouth'
$resourceGroup = 'rg-<ambiente>'
$prefix = '<prefixo-unico>'
$repository = 'JAyres88/<REPOSITORIO>'
$product = '<produto>'
az account set --subscription $subscription
```

## 2. Grupo, provedores e registro de imagens

```powershell
az group create --name $resourceGroup --location $location
'Microsoft.App','Microsoft.ContainerRegistry','Microsoft.Sql','Microsoft.Network','Microsoft.KeyVault' | ForEach-Object { az provider register --namespace $_ }
az acr create --resource-group $resourceGroup --name "acr$prefix" --sku Basic
az keyvault create --resource-group $resourceGroup --name "kv$prefix" --location $location
```

## 3. Rede e SQL privado

```powershell
az network vnet create --resource-group $resourceGroup --name "vnet-$product" --address-prefixes 10.80.0.0/16 --subnet-name snet-containerapps --subnet-prefixes 10.80.0.0/23
az network vnet subnet update --resource-group $resourceGroup --vnet-name "vnet-$product" --name snet-containerapps --delegations Microsoft.App/environments
az network vnet subnet create --resource-group $resourceGroup --vnet-name "vnet-$product" --name snet-private-endpoints --address-prefixes 10.80.2.0/24 --disable-private-endpoint-network-policies true
az sql server create --resource-group $resourceGroup --name "sql$prefix" --location $location --admin-user '<ADMIN_SQL>' --admin-password '<SENHA_ARMAZENADA_COM_SEGURANCA>'
az sql db create --resource-group $resourceGroup --server "sql$prefix" --name "$product-teste" --edition GeneralPurpose --family Gen5 --capacity 1 --compute-model Serverless --min-capacity 0.5 --auto-pause-delay 60 --backup-storage-redundancy Local
az network private-dns zone create --resource-group $resourceGroup --name privatelink.database.windows.net
az network private-dns link vnet create --resource-group $resourceGroup --zone-name privatelink.database.windows.net --name "link-$product" --virtual-network "vnet-$product" --registration-enabled false
az network private-endpoint create --resource-group $resourceGroup --name "pe-sql-$product" --vnet-name "vnet-$product" --subnet snet-private-endpoints --private-connection-resource-id $(az sql server show --resource-group $resourceGroup --name "sql$prefix" --query id --output tsv) --group-id sqlServer --connection-name "peconn-sql-$product"
az sql server update --resource-group $resourceGroup --name "sql$prefix" --enable-public-network false
```

Crie a cadeia de conexão somente no Key Vault. A senha deve ser obtida de um segredo existente ou digitada localmente, nunca colocada em texto no comando versionado.

```powershell
az keyvault secret set --vault-name "kv$prefix" --name "$product-sql-connection" --value '<CADEIA_DE_CONEXAO>'
```

## 4. Ambiente e identidade

```powershell
az identity create --resource-group $resourceGroup --name "id-$product-deploy"
$identityId = az identity show --resource-group $resourceGroup --name "id-$product-deploy" --query id --output tsv
$principalId = az identity show --resource-group $resourceGroup --name "id-$product-deploy" --query principalId --output tsv
$acrId = az acr show --resource-group $resourceGroup --name "acr$prefix" --query id --output tsv
az role assignment create --assignee-object-id $principalId --assignee-principal-type ServicePrincipal --role AcrPull --scope $acrId
az role assignment create --assignee-object-id $principalId --assignee-principal-type ServicePrincipal --role AcrPush --scope $acrId
az containerapp env create --resource-group $resourceGroup --name "cae-$product-teste" --location $location --infrastructure-subnet-resource-id $(az network vnet subnet show --resource-group $resourceGroup --vnet-name "vnet-$product" --name snet-containerapps --query id --output tsv)
```

Conceda à identidade somente leitura dos segredos necessários no Key Vault. Use RBAC ou política de acesso conforme o cofre já estiver configurado.

## 5. Aplicativos e trabalho de migração

Crie primeiro os recursos com uma imagem temporária. Depois o pipeline substituirá as imagens pelos pacotes privados.

```powershell
$environment = "cae-$product-teste"
$secret = "$product-sql-connection=keyvaultref:https://kv$prefix.vault.azure.net/secrets/$product-sql-connection,identityref:$identityId"
az containerapp create --resource-group $resourceGroup --name "ca-$product-api-teste" --environment $environment --image mcr.microsoft.com/k8se/quickstart:latest --ingress internal --target-port 8080 --min-replicas 0 --max-replicas 1 --user-assigned $identityId --secrets $secret --env-vars ConnectionStrings__DefaultConnection="secretref:$product-sql-connection"
az containerapp create --resource-group $resourceGroup --name "ca-$product-web-teste" --environment $environment --image mcr.microsoft.com/k8se/quickstart:latest --ingress external --target-port 8080 --min-replicas 0 --max-replicas 1 --user-assigned $identityId
az containerapp job create --resource-group $resourceGroup --name "job-$product-migrations-teste" --environment $environment --trigger-type Manual --replica-timeout 1800 --replica-retry-limit 1 --parallelism 1 --replica-completion-count 1 --image mcr.microsoft.com/k8se/quickstart:latest --mi-user-assigned $identityId --secrets $secret --env-vars ConnectionStrings__DefaultConnection="secretref:$product-sql-connection"
```

Associe os três componentes ao ACR:

```powershell
$registry = "acr$prefix.azurecr.io"
az containerapp registry set --resource-group $resourceGroup --name "ca-$product-api-teste" --server $registry --identity $identityId
az containerapp registry set --resource-group $resourceGroup --name "ca-$product-web-teste" --server $registry --identity $identityId
az containerapp job registry set --resource-group $resourceGroup --name "job-$product-migrations-teste" --server $registry --identity $identityId
```

## 6. GitHub Actions por OpenID Connect

Crie o ambiente e as variáveis não sigilosas. O identificador numérico do repositório é obtido pela API do GitHub.

```powershell
$repoId = gh api "repos/$repository" --jq .id
gh api --method PUT "repos/$repository/environments/teste" | Out-Null
az identity federated-credential create --resource-group $resourceGroup --identity-name "id-$product-deploy" --name "github-$product-teste" --issuer https://token.actions.githubusercontent.com --subject "repo:JAyres88@22259448/<REPOSITORIO>@$repoId`:environment:teste" --audiences api://AzureADTokenExchange
gh variable set AZURE_CLIENT_ID --repo $repository --env teste --body $(az identity show --resource-group $resourceGroup --name "id-$product-deploy" --query clientId --output tsv)
gh variable set AZURE_TENANT_ID --repo $repository --env teste --body '<TENANT_ID>'
gh variable set AZURE_SUBSCRIPTION_ID --repo $repository --env teste --body $subscription
gh variable set AZURE_ACR_NAME --repo $repository --env teste --body "acr$prefix"
```

O workflow deve usar `azure/login@v2`, `id-token: write`, montar imagens identificadas por `GITHUB_SHA`, atualizar o trabalho de migração, aguardar `Succeeded` e só então atualizar API e portal.

## 7. Verificação

```powershell
az containerapp job execution list --resource-group $resourceGroup --name "job-$product-migrations-teste" --output table
az containerapp list --resource-group $resourceGroup --query '[].{Nome:name,Publico:properties.configuration.ingress.external,Endereco:properties.configuration.ingress.fqdn}' --output table
Invoke-WebRequest -UseBasicParsing "https://<FQDN_DO_PORTAL>/" | Select-Object StatusCode
```

O sucesso exige migração concluída, API interna saudável, portal retornando HTTP 200 e SQL inacessível pela rede pública.
