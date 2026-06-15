# ⚙️ Guia de Configuração

Este guia descreve **passo a passo** como configurar e ajustar o dashboard `Controle Disparos SocketLabs` para o seu ambiente.

---

## 1. Configurar a Fonte de Dados (Planilha Google Sheets)

### 1.1 Publicar a planilha como `.xlsx`

1. Abra a planilha no Google Sheets.
2. Vá em **Arquivo → Compartilhar → Publicar na web**.
3. Em "Tipo de conteúdo", selecione a aba relevante (ou "Documento inteiro").
4. Em "Tipo de arquivo", selecione **Microsoft Excel (.xlsx)**.
5. Clique em **Publicar** e copie o link gerado (deve terminar com `pub?output=xlsx`).

> ⚠️ **Atenção**: a planilha precisa permanecer com acesso "Qualquer pessoa com o link pode visualizar" para que o `fetch()` funcione sem autenticação.

### 1.2 Atualizar a constante no código

No arquivo `index.html`, localize (próximo à linha 469):

```js
const URL_PLANILHA_FIXA = 'https://docs.google.com/spreadsheets/d/e/SEU_ID/pub?output=xlsx';
```

Substitua pelo novo link publicado. Não é necessário nenhum outro ajuste — o `window.onload` já carrega esse link automaticamente.

---

## 2. Estrutura Mínima da Planilha

Garanta que a planilha publicada contenha (ou possa ser localizada como) as abas:

| Aba | Colunas mínimas obrigatórias |
|---|---|
| `Geral` | `Mes`, `Realizado`, `UsoGeral`, `Última atualização` |
| `Historico` | `Mes`, `v25`, `v26` |
| `Campanhas` | `Dia`, `Base`, `Status` |

> Veja a especificação completa em [`MODELO_DADOS.md`](MODELO_DADOS.md).

🟢 **Recomendação**: mantenha os nomes das abas exatamente como `Geral`, `Historico` e `Campanhas` (sem acentos) para evitar depender da lógica de detecção tolerante (`getSheet`), cujo conjunto completo de aliases não está 100% documentado.

---

## 3. Ajustar Metas e Capacidade

No início do bloco `<script>` (próximo à linha 463–464):

```js
const META_ALUNOS = 11700000;     // Meta mensal CRM Alunos (11.7M)
const CAPACIDADE_MAX = 20000000;  // Capacidade compartilhada mensal (20M)
```

### Impactos ao alterar `META_ALUNOS`:
- KPI "Realizado" e "% da Meta" (`#kpi-pct-meta`)
- KPI "Saldo Disponível" (`#kpi-saldo`)
- Cálculo de "Peso" nas tabelas de campanhas (`Base / META_ALUNOS * 100`)
- Linha de meta no gráfico de sazonalidade (texto fixo `"META 11.7M"` — **precisa ser editado manualmente** em dois locais: HTML estático `<span class="meta-label">META: 11.7M</span>` e no JS `chartMain.innerHTML = ... META 11.7M ...`)
- Cálculo de status diário (`calcularStatusMetaTemporal`)

> 🟢 **Recomendação**: ao alterar a meta, atualizar também os textos fixos "11.7M" no HTML/JS (não são gerados dinamicamente a partir da constante).

### Impactos ao alterar `CAPACIDADE_MAX`:
- KPIs "Uso Alunos" e "Uso Geral" (`#kpi-uso-alunos`, `#kpi-uso-geral`, `#kpi-uso-geral-pct`)
- Visão anual: `metaPacote = CAPACIDADE_MAX * 12`
- Gráfico "Foco" (`hFactor = CAPACIDADE_MAX` ou `CAPACIDADE_MAX * 8`)
- Texto fixo `"Base cálculo: 20M"` no HTML (também precisa atualização manual)

---

## 4. Configurar o Tema Padrão (Claro/Escuro)

Por padrão, o dashboard:
1. Verifica `localStorage.theme` (se o usuário já escolheu antes).
2. Se não houver preferência salva, usa `prefers-color-scheme` do sistema operacional/navegador.

Não há configuração manual de "tema padrão fixo" — para forçar um tema específico independente da preferência do usuário, seria necessário ajustar `applySavedTheme()`.

🟡 **Hipótese**: não há indicação no código de necessidade de tema fixo; o comportamento atual (seguir preferência do usuário/sistema) parece intencional.

---

## 5. Reativar Importação Manual de Excel (opcional)

Por padrão, os botões de importação manual (`#btnImport`) e atualização (`#btnUpdate`), bem como o campo de URL (`#sheetUrl`), estão ocultos (`hidden`).

Para reativar:

1. Localize os elementos no HTML (próximo às linhas 265–280):
   ```html
   <div hidden class="relative w-full md:w-64"> ... <input id="sheetUrl" ...> ... </div>
   <div hidden> <button hidden id="btnUpdate" ...> ATUALIZAR </button> </div>
   <label id="btnImport" hidden ...> IMPORTAR ... </label>
   ```
2. Remova os atributos `hidden` desejados.

🟢 **Recomendação**: ao reativar `#btnImport`, lembre-se que o fluxo de upload manual usa **apenas a primeira aba** da planilha local e espera todas as colunas (`Mes`, `v25`, `v26`, `Status`, `Realizado`, etc.) na mesma aba — diferente do fluxo via `URL_PLANILHA_FIXA`, que usa 3 abas separadas. Veja [`MODELO_DADOS.md`](MODELO_DADOS.md) e [`FUNCOES.md`](FUNCOES.md).

---

## 6. Publicar/Hospedar o Dashboard

### GitHub Pages (recomendado para uso simples)
1. Faça push do repositório.
2. Settings → Pages → Source: branch `main`, pasta `/root`.
3. Aguarde alguns minutos e acesse a URL gerada.

### Hospedagem interna
Como é um arquivo HTML estático autocontido (CSS e JS inline, dependências via CDN), pode ser:
- Servido por qualquer servidor web (Nginx, Apache, IIS).
- Embutido em um iframe dentro de uma intranet/SharePoint.
- Aberto localmente via `file://` (algumas restrições de CORS podem afetar o `fetch()` da planilha, dependendo do navegador).

> 🟡 **Hipótese**: ao abrir via `file://` (sem servidor), alguns navegadores podem bloquear o `fetch()` por política de CORS/segurança local. 🟢 **Recomendação**: para testes locais, preferir servir via um servidor HTTP simples (`python -m http.server`, extensão "Live Server", etc.).

---

## 7. Checklist Rápido de Configuração

| Item | Verificar |
|---|---|
| ☐ | Planilha publicada como `.xlsx` com acesso público |
| ☐ | `URL_PLANILHA_FIXA` atualizada no `index.html` |
| ☐ | Abas `Geral`, `Historico`, `Campanhas` presentes com colunas mínimas |
| ☐ | `META_ALUNOS` e `CAPACIDADE_MAX` revisados (e textos fixos "11.7M"/"20M" atualizados, se necessário) |
| ☐ | Hospedagem definida (GitHub Pages, servidor interno, etc.) |
| ☐ | Teste em modo claro e escuro |
| ☐ | Teste com mês filtrado e visão anual (sem filtro) |
