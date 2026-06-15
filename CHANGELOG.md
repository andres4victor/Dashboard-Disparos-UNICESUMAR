# 📝 Changelog

Histórico de versões e alterações do dashboard **Controle Disparos SocketLabs — Unicesumar** e de sua documentação.

O formato segue aproximadamente [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/), adaptado ao contexto deste projeto (arquivo único, sem versionamento semântico formal até o momento).

---

## [Documentação inicial] — 2026-06-15

### Adicionado
- 🟢 Criação do repositório de documentação `controle-disparos-socketlabs`.
- 🟢 `README.md` com visão geral, objetivo, stack técnica e instruções de uso/publicação.
- 🟢 `docs/ARQUITETURA.md` — arquitetura técnica, fluxo de execução e estado global.
- 🟢 `docs/MODELO_DADOS.md` — especificação das abas/colunas esperadas na planilha (`Geral`, `Historico`, `Campanhas`) e fórmulas de KPI.
- 🟢 `docs/FUNCOES.md` — referência completa de todas as funções JavaScript do `index.html`.
- 🟢 `docs/GUIA_CONFIGURACAO.md` — passo a passo de configuração (planilha, metas, tema, hospedagem).

### Observações sobre o estado do código analisado (`index.html`)
- 🔵 Identificadas **4 funções duplicadas** no script (`getDaysInMonth`, `formatPctBR`, `formatSignedPctBR`, `calcularStatusMetaTemporal`) — sem impacto funcional, mas redundantes.
- 🔵 Identificada inconsistência de nomenclatura de IDs HTML: a tabela "Enviados" usa `#table-pausados`/`#sum-pausados` (nomes legados, deveriam ser `#table-enviados`/`#sum-enviados`).
- 🔵 Botões de importação manual (`#btnImport`), atualização (`#btnUpdate`) e campo de link (`#sheetUrl`) estão atualmente ocultos (`hidden`); a fonte de dados ativa é exclusivamente `URL_PLANILHA_FIXA` via `window.onload`.
- 🔵 Existe um segundo `window.onload` (modo "link configurável via localStorage") e a função `generateMockData()`, ambos preservados no código mas não utilizados no fluxo ativo.

---

## Próximas iterações planejadas

- 🟡 Renomear IDs legados (`#table-pausados` → `#table-enviados`, `#sum-pausados` → `#sum-enviados`) — requer ajuste simultâneo no HTML e no JS (`renderDashboard`).
- 🟡 Remover bloco de funções duplicadas (manter apenas a segunda definição, idêntica).
- 🟡 Avaliar necessidade de reativação dos controles de importação manual/atualização (`#btnImport`, `#btnUpdate`, `#sheetUrl`).
- 🟡 Validar e documentar de forma definitiva os aliases de nomes de aba aceitos por `getSheet()` em `fetchOnlineData`.

> Itens desta seção são marcados como 🟡 (hipótese/planejamento) até serem efetivamente executados e confirmados, quando passarão a novas entradas neste changelog com status 🟢/🔵.

---

## Template para novas entradas

```md
## [Nome/Versão] — AAAA-MM-DD

### Adicionado
- 🟢 ...

### Alterado
- 🔵 ...

### Corrigido
- 🔵 ...

### Removido
- 🔵 ...
```
