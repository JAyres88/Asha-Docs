# Natureza do Tipo de Saldo

## Objetivo

Classificar cada tipo de saldo conforme seu significado operacional, sem confundir quantidades físicas, comprometidas e apenas informativas.

## Escopo

- Adicionar as naturezas `Fisico`, `Comprometido` e `Informativo` ao cadastro de Tipo de Saldo.
- Expor a natureza nos contratos da API, Swagger e frontend.
- Exibir a natureza nas consultas e no cadastro de tipos de saldo.
- Impedir combinações que façam um saldo informativo participar de totais físicos ou disponíveis.
- Preservar os tipos de saldo e valores já gravados.

## Critérios de aceite

- Todo Tipo de Saldo possui uma natureza válida.
- Saldos informativos continuam movimentáveis e consultáveis.
- Saldos informativos não entram no total físico nem no disponível.
- A migration atribui uma natureza compatível aos registros existentes.
- Testes cobrem criação, edição, consulta e compatibilidade da migration.

## Ordem

Executar antes das regras configuráveis de movimentação por Tipo de Documento.
