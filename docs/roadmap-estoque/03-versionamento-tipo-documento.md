# Versionamento do Tipo de Documento

## Objetivo

Preservar regras operacionais historicamente sem duplicá-las em cada documento. Cada documento referencia a versão publicada utilizada em sua criação.

## Funcionamento

- Um Tipo de Documento possui versões numeradas.
- Alterar sua configuração encerra a versão vigente e publica uma nova.
- As regras de saldo pertencem à versão e tornam-se imutáveis historicamente.
- Documentos novos apontam para a versão vigente.
- Documentos existentes continuam vinculados à versão original.
- Documentos anteriores ao versionamento permanecem identificados como legados.

## Dados versionados

- finalidade e geração de saldo documental;
- operação e dimensões de estoque mantidas para compatibilidade;
- exigência de endereço e permissão de processamento;
- regras ordenadas de soma e subtração;
- locais, tipos de saldo e permissões de alteração.

## Critérios de aceite

- A primeira publicação cria a versão 1.
- Uma alteração encerra a vigente e cria a próxima versão.
- Apenas uma versão pode estar vigente por Tipo de Documento.
- O documento informa a versão utilizada.
- Alterar o cadastro não modifica a configuração histórica.
- Tipos antigos recebem sua primeira versão automaticamente quando utilizados.

## Dependência

Implementar depois de **Regras de Movimentação por Tipo de Documento** e antes de **Processamento das Regras de Saldo**.
