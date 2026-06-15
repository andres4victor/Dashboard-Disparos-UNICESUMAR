# 🧩 Referência de Funções (FUNCOES.md)

Documentação de todas as funções JavaScript presentes no `index.html`, organizadas por categoria. Para cada função: assinatura, propósito, parâmetros, retorno e observações relevantes.

---

## 1. Utilitários Gerais

### `getDaysInMonth(date)`
- **Propósito**: retorna o número de dias do mês de uma data.
- **Parâmetro**: `date` (`Date`)
- **Retorno**: `number` — quantidade de dias do mês.
- 🔵 **Fato observado**: esta função está **declarada duas vezes** no arquivo (linhas ~477 e ~614), com implementação idêntica. Não causa erro (a segunda sobrescreve a primeira), mas é código duplicado.

---

### `formatPctBR(value)`
- **Propósito**: formata um número como percentual no padrão brasileiro (`vírgula` decimal).
- **Parâmetro**: `value` (`number`)
- **Retorno**: `string` — ex: `"73,45%"`
- 🔵 **Fato observado**: também declarada duas vezes (linhas ~481 e ~618), idêntica.

---

### `formatSignedPctBR(value)`
- **Propósito**: formata percentual com sinal explícito (`+`/`-`), padrão BR.
- **Parâmetro**: `value` (`number`)
- **Retorno**: `string` — ex: `"+2,30%"`, `"-1,05%"`, `"0,00%"`
- 🔵 **Fato observado**: também duplicada (linhas ~485 e ~622).

---

### `formatMillions(value)`
- **Propósito**: formata um valor numérico em escala compacta (M/K).
- **Parâmetro**: `value` (`number`)
- **Retorno**: `string`
  - `>= 1.000.000` → `"X.XXM"`
  - `>= 1.000` → `"X.XK"`
  - menor → valor bruto como string
- **Uso**: KPIs (`#kpi-realizado`, `#kpi-projetado`, `#kpi-uso-geral`) e somatórios das tabelas.

---

### `excelValueToDate(value)`
- **Propósito**: converter valores de data vindos da planilha (serial numérico, string `dd/mm[/aaaa]` ou `Date`) em objeto `Date` do JS.
- **Parâmetro**: `value` (`number | string | Date | null`)
- **Retorno**: `Date | null`
- **Lógica**:
  - `Date` → retorna como está
  - `number` → calcula a partir da data-base `30/12/1899` (padrão Excel/Sheets)
  - `string` com `/` → faz split e monta `new Date(ano, mes-1, dia)`; se ano ausente, usa ano atual
  - outro → `null`
- **Usada por**: `formatDatePtBr`, `autoFilterByUpdateDate`, `calcularStatusMetaTemporal`, filtro de campanhas por mês.

---

### `formatDatePtBr(value)`
- **Propósito**: formatar uma data (em qualquer formato aceito por `excelValueToDate`) no padrão `dd/mm/aaaa`.
- **Parâmetro**: `value`
- **Retorno**: `string` (vazio se data inválida)
- **Uso**: header (`#lastUpdateDate`), coluna "Dia" das tabelas de campanhas.

---

## 2. Cálculo de Metas

### `calcularStatusMetaTemporal(rawUpdate, realizado, metaMensal)`
- **Propósito**: calcular se o ritmo de envio está dentro ou fora da meta, com base na data da última atualização.
- **Parâmetros**:
  - `rawUpdate`: valor bruto da data de última atualização (qualquer formato aceito por `excelValueToDate`)
  - `realizado`: valor total já disparado no mês (`number`)
  - `metaMensal`: meta mensal de referência (`number`, normalmente `META_ALUNOS`)
- **Retorno**: `object | null`
  ```js
  {
    updateDate,       // Date
    diaAtualizacao,   // number (dia do mês da atualização)
    diasNoMes,        // number (total de dias do mês)
    percentualIdeal,  // number (% esperado até hoje)
    percentualAtual,  // number (% real alcançado)
    diferenca,        // number (ideal - atual)
    dentroDaMeta      // boolean (diferenca >= 0)
  }
  ```
  Retorna `null` se `rawUpdate` for inválido ou `metaMensal` for `0`/`falsy`.
- 🔵 **Fato observado**: também duplicada no arquivo (linhas ~492 e ~629), implementação idêntica.

---

## 3. Carregamento e Inicialização

### `window.onload` (handler anônimo)
- **Propósito**: ponto de entrada da aplicação.
- **Ações**:
  1. Preenche `#sheetUrl` com `URL_PLANILHA_FIXA`.
  2. Chama `fetchOnlineData(true)`.
  3. Chama `applySavedTheme()`.
- 🔵 **Fato observado**: existe um segundo `window.onload` **comentado** no código (modo "link online" via `localStorage`), mantido como referência/fallback histórico — não é executado.

---

### `generateMockData()`
- **Propósito**: gerar dados fictícios para testes/demonstração, sem depender de planilha externa.
- **Parâmetros**: nenhum
- **Retorno**: `void` (atribui a `rawData`, chama `autoFilterByUpdateDate` e `renderDashboard`)
- **Estrutura gerada**: `Config`, `Historico` (12 meses com `v25`/`v26` aleatórios), `Campanhas` (2 itens de exemplo), `Geral` (12 meses com valores aleatórios para Jan–Abr).
- 🟡 **Hipótese**: atualmente não é chamada em nenhum fluxo ativo (o `window.onload` ativo usa `fetchOnlineData`), permanecendo como utilitário para debug manual via console.

---

### `autoFilterByUpdateDate(data)`
- **Propósito**: definir automaticamente `selectedMonth` com base no mês da "Última atualização".
- **Parâmetro**: `data` (objeto `rawData`)
- **Retorno**: `void` (efeito colateral: define `selectedMonth`)
- **Lógica**:
  1. Procura em `data.Config` a primeira linha com `'Última atualização'` preenchida.
  2. Converte para `Date` via `excelValueToDate`.
  3. Se válido, define `selectedMonth = meses[d.getMonth()]`.
  4. Se nada for encontrado, define `selectedMonth = 'Jan'` (fallback).

---

### `fetchOnlineData(isInitial = false)`
- **Propósito**: buscar a planilha publicada, parsear e popular `rawData`.
- **Parâmetro**: `isInitial` (`boolean`) — controla se mensagens de erro/sucesso (toasts) são exibidas (suprimidas na carga inicial).
- **Retorno**: `Promise<void>`
- **Fluxo**:
  1. Lê URL de `#sheetUrl`. Se vazio, retorna (mostra toast de erro se `!isInitial`).
  2. Desabilita `#btnUpdate` e mostra spinner "SINCRONIZANDO...".
  3. `fetch(url)` → `arrayBuffer()` → `XLSX.read(arrayBuffer, {type:'array'})`.
  4. Usa `getSheet(names)` (helper interno) para localizar as abas `Geral`/`Historico`/`Campanhas` por nome, tolerando variações.
  5. Monta `rawData = { Config: geralRows, Historico, Campanhas, Geral: geralRows }`.
  6. Salva a URL em `localStorage` (`STORAGE_KEY`).
  7. Chama `autoFilterByUpdateDate(rawData)` e `renderDashboard(rawData)`.
  8. Em caso de erro: define `#lastUpdateDate` = "Erro no link" e mostra toast de erro (se `!isInitial`).
  9. `finally`: reabilita `#btnUpdate` e restaura o texto do botão.
- 🟡 **Hipótese**: as linhas 699–717 (lógica interna de `getSheet` e leitura das abas) não foram totalmente inspecionadas nesta documentação — recomenda-se revisão pontual caso a planilha use nomes de aba não previstos. 🟢 **Recomendação**: documentar explicitamente, em `GUIA_CONFIGURACAO.md` ou via comentário no código, a lista exata de nomes de aba aceitos.

---

### Handler de upload manual — `#uploadExcel` `change`
- **Propósito**: importar dados de um arquivo `.xlsx`/`.xls` local.
- **Fluxo**:
  1. Lê o arquivo via `FileReader.readAsBinaryString`.
  2. `XLSX.read(data, {type:'binary'})` → pega a **primeira aba** do workbook.
  3. Constrói `rawData` de forma simplificada:
     - `Config`: todas as linhas
     - `Historico`: linhas com campo `Mes`
     - `Campanhas`: linhas com campo `Status`
     - `Geral`: linhas com campo `Realizado`
  4. Chama `autoFilterByUpdateDate` e `renderDashboard`.
  5. Mostra toast de sucesso ou erro.
- 🔵 **Fato observado**: este modo usa **uma única aba** da planilha (a primeira), diferente do `fetchOnlineData` que procura abas específicas por nome. Os dados devem conter todas as colunas necessárias (`Mes`, `v25`, `v26`, `Status`, `Realizado`, etc.) na mesma planilha.

---

## 4. Renderização

### `renderDashboard(data)`
- **Propósito**: função central que atualiza **toda a interface** com base em `data` e `selectedMonth`.
- **Parâmetro**: `data` (objeto `rawData`)
- **Retorno**: `void`
- Ver detalhamento completo em [`ARQUITETURA.md`](ARQUITETURA.md#renderização-renderdashboard) e [`MODELO_DADOS.md`](MODELO_DADOS.md).
- **Sub-blocos principais** (em ordem):
  1. Última atualização → `#lastUpdateDate`
  2. Banner de filtro + labels `.current-filter-label`
  3. Gráfico de sazonalidade (`#mainChart`)
  4. Tabelas de campanhas (Agendados/Enviados/Pausados/Cancelados) + somatórios
  5. KPIs de meta (Realizado, Projetado, Saldo)
  6. Status diário da meta (`#meta-status-text`)
  7. KPIs de capacidade compartilhada (Uso Alunos, Uso Geral)
  8. Gráfico "Foco" (`#comparativoMarco`)

---

### `renderEmptyTableState(tbody, message)`
- **Propósito**: exibir uma linha de "estado vazio" em uma tabela sem dados para o período filtrado.
- **Parâmetros**:
  - `tbody` (`HTMLElement`)
  - `message` (`string`, padrão: `'Nenhum dado encontrado neste período.'`)
- **Retorno**: `void`
- **Lógica**: só insere a linha se `tbody.children.length === 0` (ou seja, se nenhuma campanha foi inserida para aquele período/tabela).

---

### `selectMonth(month)`
- **Propósito**: alternar o filtro de mês ao clicar em uma barra do gráfico de sazonalidade.
- **Parâmetro**: `month` (`string`, ex: `'Mar'`)
- **Retorno**: `void`
- **Lógica**: se `month` já é o mês selecionado, remove o filtro (`null`); caso contrário, define `selectedMonth = month`. Em seguida, chama `renderDashboard(rawData)`.

---

### `clearSavedLink()`
- **Propósito**: remover o link salvo da planilha do `localStorage` e limpar o campo `#sheetUrl`.
- **Parâmetro**: nenhum
- **Retorno**: `void`
- **Efeito**: exibe toast de erro/aviso `"Link removido."`.
- 🟡 **Hipótese**: como o input `#sheetUrl` está atualmente `hidden` e a constante `URL_PLANILHA_FIXA` é fixa no código, esta função tem **uso limitado no fluxo atual** — permanece útil caso o modo de link configurável seja reativado.

---

## 5. Notificações (Toasts)

### `showToast(message, type = 'success')`
- **Propósito**: exibir uma notificação temporária (toast) no canto inferior direito.
- **Parâmetros**:
  - `message` (`string`)
  - `type` (`'success' | 'error'`, padrão `'success'`)
- **Retorno**: `void`
- **Lógica**:
  - Cria `<div class="toast toast-{type}">` com ícone (`fa-circle-check` ou `fa-circle-exclamation`) e o texto.
  - Adiciona classe `.show` após 10ms (transição de entrada).
  - Remove classe `.show` após 3s e remove o elemento do DOM após +300ms.

---

## 6. Tema (Claro/Escuro)

### `applySavedTheme()`
- **Propósito**: aplicar o tema salvo (`localStorage.theme`) ou, na ausência dele, a preferência do sistema (`prefers-color-scheme`).
- **Parâmetros**: nenhum
- **Retorno**: `void`
- **Lógica**:
  1. Lê `localStorage.getItem('theme')`.
  2. Verifica `window.matchMedia('(prefers-color-scheme: dark)')`.
  3. Define `shouldBeDark = saved === 'dark' || (!saved && prefersDark)`.
  4. Aplica/remove classe `.dark` em `<html>`.
  5. Alterna visibilidade dos ícones `#icon-moon` / `#icon-sun`.

---

### `toggleTheme()`
- **Propósito**: alternar manualmente entre modo claro e escuro (botão `#themeToggle`).
- **Parâmetros**: nenhum
- **Retorno**: `void`
- **Lógica**:
  1. Alterna classe `.dark` em `<html>`.
  2. Salva a escolha em `localStorage.theme`.
  3. Alterna visibilidade dos ícones de sol/lua.

---

## 📋 Resumo — Tabela de Funções

| Função | Categoria | Chamada por |
|---|---|---|
| `getDaysInMonth` | Utilitário | `calcularStatusMetaTemporal` |
| `formatPctBR` | Utilitário | `calcularStatusMetaTemporal` (indireto via render) |
| `formatSignedPctBR` | Utilitário | `renderDashboard` (status da meta) |
| `formatMillions` | Utilitário | `renderDashboard` (KPIs e somatórios) |
| `excelValueToDate` | Utilitário/Data | `formatDatePtBr`, `autoFilterByUpdateDate`, `calcularStatusMetaTemporal`, `renderDashboard` |
| `formatDatePtBr` | Utilitário/Data | `renderDashboard` |
| `calcularStatusMetaTemporal` | Meta | `renderDashboard` |
| `generateMockData` | Dados | Não chamada no fluxo ativo (utilitário de debug) |
| `autoFilterByUpdateDate` | Dados | `window.onload` (via `fetchOnlineData`), upload manual |
| `fetchOnlineData` | Dados | `window.onload`, botão `#btnUpdate` (oculto) |
| Handler `#uploadExcel` | Dados | Evento `change` do input de upload |
| `renderDashboard` | Render | `fetchOnlineData`, upload manual, `selectMonth`, `generateMockData` |
| `renderEmptyTableState` | Render | `renderDashboard` |
| `selectMonth` | Render/Interação | `onclick` das barras do gráfico de sazonalidade |
| `clearSavedLink` | Configuração | botão "x" do campo `#sheetUrl` (oculto) |
| `showToast` | UI/Feedback | `fetchOnlineData`, upload manual, `clearSavedLink` |
| `applySavedTheme` | Tema | `window.onload` |
| `toggleTheme` | Tema | `#themeToggle` (click) |

---

## ⚠️ Observações Gerais de Código

- 🔵 **Fato observado**: quatro funções estão **duplicadas** no arquivo (`getDaysInMonth`, `formatPctBR`, `formatSignedPctBR`, `calcularStatusMetaTemporal`) — primeira definição entre as linhas ~477–512, segunda entre ~614–649. JavaScript permite redeclaração de `function` no mesmo escopo (a segunda sobrescreve a primeira sem erro), mas é redundância de código.
- 🟢 **Recomendação**: remover o primeiro bloco duplicado (linhas ~474–512) em uma futura limpeza, mantendo apenas a segunda definição (idêntica).
- 🔵 **Fato observado**: existe um segundo `window.onload` comentado (modo de link configurável via `localStorage`), preservado como referência histórica.
