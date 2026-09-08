# Regras de Movimentação por Tipo de Documento

## Objetivo

Substituir a configuração limitada de origem e destino por uma coleção ordenada de lançamentos de saldo.

## Escopo

- Criar a associação entre Tipo de Documento, Tipo de Saldo, operação e local.
- Permitir as operações `Somar` e `Subtrair`.
- Exibir um grid editável no cadastro de Tipo de Documento.
- Permitir múltiplos lançamentos para o mesmo documento.
- Validar que o tipo de saldo esteja habilitado no local selecionado.
- Manter temporariamente a leitura da configuração anterior para compatibilidade.

## Exemplo

| Tipo de saldo | Operação |
| --- | --- |
| Disponível | Subtrair |
| Pedido de venda | Somar |
| Empenhado | Somar |

## Critérios de aceite

- O grid permite incluir, ordenar e remover regras.
- Não são permitidas regras duplicadas ou incompletas.
- O cadastro informa claramente natureza, saldo, operação e local.
- API, Swagger e frontend utilizam o mesmo contrato.
- Testes cobrem regras simples e múltiplas.

## Dependência

Implementar depois de **Natureza do Tipo de Saldo**.
