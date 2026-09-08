# Normalização do layout responsivo

## Objetivo

Estabelecer uma escala visual única para o frontend do Main, reduzindo menus, textos e espaçamentos excessivos sem comprometer legibilidade, acessibilidade ou áreas de toque em telas menores.

## Escala base

- Texto padrão: `12px`.
- Texto auxiliar: `10px`.
- Título de página: `17px`.
- Barra superior: `42px`.
- Menu lateral desktop: `210px`.
- Controle de formulário desktop: `34px`.
- Linha de grid desktop: `44px`.
- Espaçamento de página: entre `16px` e `24px`.

As medidas são declaradas como tokens no `app.css`. Novos componentes devem reutilizar esses tokens em vez de criar dimensões arbitrárias.

## Comportamento responsivo

### Desktop

- Menu lateral completo e mais compacto.
- Cabeçalhos, barras de comando e filtros com menor altura.
- Grids densos, mantendo leitura e seleção de linhas.
- Formulários com controles e intervalos uniformes.
- Modais limitados à área visível.

### Até 760 px

- Menu lateral reduzido a ícones.
- Comandos secundários indisponíveis são ocultados.
- Barras de comando permitem rolagem horizontal.
- Controles recebem altura mínima de `38px` para toque.
- Grids preservam rolagem horizontal em vez de comprimir colunas.
- Modais ocupam toda a tela.
- Seleção de módulos usa uma coluna.

## Componentes normalizados

- Barra global e menus de perfil/configurações.
- Navegação lateral e seletor de módulo.
- Cabeçalhos de páginas.
- Barras de comando, filtros e pesquisa.
- Formulários e campos de consulta contextual.
- Grids de registros e grids internos.
- Seções de documento, estoque e endereços.
- Modais e diálogos de pesquisa.
- Tela de seleção de módulos.

## Regras para evolução

- Preferir os tokens `--ui-*` definidos globalmente.
- Evitar alturas fixas maiores que o necessário em desktop.
- Não reduzir controles móveis abaixo da área de toque definida.
- Não remover títulos de coluna para economizar espaço.
- Em grids largos, usar rolagem horizontal.
- Manter `aria-label` quando um rótulo visual for omitido.
- Validar novos layouts em desktop, tablet e largura móvel.
