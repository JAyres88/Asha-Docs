# Renomeação da plataforma para Asha

A marca e os repositórios vigentes são Asha Portal, Asha Gestão, Asha Ponto de Venda, Asha Identity, Asha Integração e Asha Onboarding & Deploy. Os projetos .NET, namespaces, assemblies, arquivos de interface, mensagens e nomes das imagens utilizam Asha. A documentação fica em Asha-Docs.

## Compatibilidade da instalação existente

A renomeação não cria outra instalação nem substitui os dados do cliente. Permanecem como identificadores de compatibilidade: nomes físicos dos bancos existentes, IDs de migração já aplicados, IDs/audiences/scopes OAuth, chave de módulo `MainDDD`, identidades de usuários, IDs de instalação/tenant, nomes das filas RabbitMQ e recursos Docker já vinculados a volumes e ao túnel. Esses valores são contratos persistidos, não nomes de apresentação. Alterá-los por substituição textual criaria bancos, usuários ou filas paralelos.

A stack existente mantém seu identificador `main`, a rede `main-provider` e os nomes de serviços para preservar volumes, resolução interna e o destino `main_gateway` configurado no Cloudflare. As novas imagens são `asha-gestao`, `asha-pdv-api`, `asha-pdv-web`, `asha-portal-api`, `asha-portal-web`, `asha-identity`, `asha-integration`, `asha-integration-web`, `asha-onboarding-api` e `asha-onboarding-worker`. Os endereços públicos em `sysgen.win` continuam os mesmos.

A migração `20260926210000_RenameDefaultBrandToAsha` atualiza somente configurações visuais que ainda correspondem aos nomes e abreviações padrão antigos. Nomes personalizados e logotipos não são sobrescritos. Credenciais e permissões não são recriadas. O cookie de sessão muda de nome; uma sessão anterior pode precisar entrar novamente.

Os diretórios de trabalho administrados pelo ambiente de desenvolvimento mantêm seus caminhos. Dentro deles, os caminhos das soluções e projetos foram atualizados.
