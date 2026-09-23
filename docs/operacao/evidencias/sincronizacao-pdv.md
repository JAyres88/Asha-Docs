# Evidências — sincronização de cadastros do PDV

## 10 de setembro de 2026

A PR [#48 do MainPDV](https://github.com/JAyres88/MainPDV/pull/48) contém a preparação da réplica operacional do PDV.

### Verificações realizadas

| Evidência | Resultado |
|---|---|
| Compilação da API do MainPDV | Aprovada, sem avisos ou erros |
| Compilação do cliente web do MainPDV | Aprovada, sem avisos ou erros |
| Rotas de escrita de cadastros mestres | Removidas |
| Métodos de escrita do cliente web | Removidos |
| Inbox de eventos internos | Implementada com deduplicação por `EventId` |

### Alterações verificadas

A API do PDV mantém somente consultas públicas para pessoas, produtos, categorias, embalagens, tabelas e preços, locais, tipos de saldo e associações. A gravação dessas entidades será exclusiva do consumidor interno de sincronização.

A rota interna `POST /api/integracao/v1/eventos` exige a chave `X-MainIntegration-Key`, registra o evento recebido e devolve sucesso sem reaplicá-lo quando o mesmo `EventId` é recebido novamente.

### Capturas de tela

A automação de captura visual desta sessão falhou antes de abrir a superfície do navegador. As evidências acima podem ser reproduzidas pela verificação da PR e pela compilação dos projetos; as capturas serão anexadas quando a automação estiver disponível.
