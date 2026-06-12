# 📊 Controle Disparos SocketLabs — Unicesumar

Dashboard estático (HTML/CSS/JS) para acompanhamento de **disparos de e-mail via SocketLabs**, controle de **meta de uso do CRM de Alunos** e **capacidade compartilhada** (CRM Alunos + CRM Mercado), com visão mensal e anual.

> Arquivo único (`index.html`) — sem backend, sem build, sem dependências de servidor. Roda em qualquer navegador ou pode ser hospedado via GitHub Pages.

---

## 🎯 Objetivo

Centralizar em um único painel visual:

- O **volume de disparos realizado** no mês/ano vs. a **meta de 11.7M** (CRM Alunos).
- O **uso compartilhado** do pacote de 20M (CRM Alunos + CRM Mercado).
- A **sazonalidade mensal** comparando 2025 x 2026.
- A lista de **campanhas/eventos** do mês (Agendados, Enviados, Pausados, Cancelados).
- O **status diário da meta** (se o ritmo de envio está dentro ou fora do esperado para o dia da atualização).

---

## 🖥️ Visão Geral da Interface

| Seção | Descrição |
|---|---|
| **Header** | Título, data da última atualização dos dados, botão de tema claro/escuro, botão de importação de Excel (oculto por padrão) |
| **Banner de Filtro** | Aparece quando um mês é selecionado no gráfico de sazonalidade; permite remover o filtro |
| **KPIs — Meta CRM Alunos** | Realizado, Projetado Total e Saldo Disponível em relação à meta de 11.7M |
| **KPIs — Capacidade Compartilhada** | Uso Alunos (%) e Uso Geral (valor + %) em relação ao pacote de 20M |
| **Gráfico de Sazonalidade Mensal** | Barras comparando 2025 (`v25`) x 2026 (`v26`) por mês, com linha de meta |
| **Gráfico "Foco"** | Comparativo do mês/ano selecionado: ano anterior, ano atual, projetado e uso geral |
| **Tabelas de Eventos** | Agendados, Enviados, Pausados e Cancelados, filtráveis por mês |

---

## 🔌 Fonte de Dados

O dashboard **carrega automaticamente** uma planilha Google Sheets publicada em formato `.xlsx`, definida na constante:

```js
const URL_PLANILHA_FIXA = 'https://docs.google.com/spreadsheets/.../pub?output=xlsx';
```

Ao abrir a página (`window.onload`), o sistema:

1. Faz `fetch()` da planilha publicada.
2. Lê o workbook com **SheetJS (XLSX.js)**.
3. Localiza as abas `Geral`, `Historico` e `Campanhas` (com tolerância a variações de nome/acentuação).
4. Monta o objeto `rawData` no formato esperado pelo dashboard.
5. Aplica o filtro automático do mês com base na data de "Última atualização".
6. Renderiza todo o painel (`renderDashboard`).

Há também um modo alternativo de **importação manual via Excel** (`#uploadExcel`), atualmente disponível como botão oculto (`hidden`), e uma função `generateMockData()` para gerar dados fictícios em ambiente de testes.

> 📄 Veja a estrutura completa das abas/colunas esperadas em [`docs/MODELO_DADOS.md`](docs/MODELO_DADOS.md).

---

## 🚀 Como usar

### Opção 1 — Abrir localmente
Basta abrir o arquivo `index.html` em qualquer navegador moderno (Chrome, Edge, Firefox). Não há necessidade de servidor.

### Opção 2 — Publicar via GitHub Pages
1. Faça push deste repositório para o GitHub.
2. Em **Settings → Pages**, selecione a branch `main` e a pasta `/ (root)`.
3. Acesse a URL gerada (ex: `https://seu-usuario.github.io/controle-disparos-socketlabs/`).

### Opção 3 — Hospedagem interna
Como é um arquivo estático, pode ser hospedado em qualquer servidor web simples (IIS, Nginx, Apache, SharePoint como página embutida, etc.).

---

## ⚙️ Configuração

| Item | Onde alterar | Descrição |
|---|---|---|
| Link da planilha | `URL_PLANILHA_FIXA` (linha ~469) | URL pública `.xlsx` do Google Sheets |
| Meta CRM Alunos | `META_ALUNOS` (linha ~463) | Valor padrão: `11.700.000` |
| Capacidade Compartilhada | `CAPACIDADE_MAX` (linha ~464) | Valor padrão: `20.000.000` |
| Tema padrão | `applySavedTheme()` | Usa preferência salva no navegador ou do sistema |

> ⚠️ Alterar essas constantes impacta **todos os cálculos de KPI, cores e gráficos**. Veja detalhes em [`docs/GUIA_CONFIGURACAO.md`](docs/GUIA_CONFIGURACAO.md).

---

## 🧱 Stack Técnica

| Camada | Tecnologia |
|---|---|
| Estrutura | HTML5 |
| Estilo | TailwindCSS (CDN) + CSS customizado (modo claro/escuro) |
| Ícones | Font Awesome 6 (CDN) |
| Leitura de planilhas | SheetJS / XLSX.js (CDN) |
| Lógica/Dados | JavaScript puro (Vanilla JS) |
| Armazenamento local | `localStorage` (tema e link de planilha) |

---

## 📁 Estrutura do Projeto

```
controle-disparos-socketlabs/
├── index.html              # Dashboard completo (HTML + CSS + JS)
├── README.md                # Este arquivo
└── docs/
    ├── ARQUITETURA.md        # Arquitetura técnica e fluxo de dados
    ├── MODELO_DADOS.md        # Estrutura esperada da planilha (abas/colunas)
    ├── FUNCOES.md             # Referência de todas as funções JavaScript
    ├── GUIA_CONFIGURACAO.md   # Como configurar metas, link e tema
    └── CHANGELOG.md           # Histórico de alterações
```

---

## 📌 Limitações Conhecidas

- 🟡 **Hipótese**: o `fetch()` da planilha pode falhar caso o link de publicação do Google Sheets seja revogado ou perca a permissão de "publicar na web".
- 🔵 **Fato observado**: o botão de importação manual de Excel (`#uploadExcel`) e o botão "Atualizar" (`#btnUpdate`) estão atualmente ocultos via atributo `hidden`.
- 🔵 **Fato observado**: não há tratamento de autenticação — a planilha precisa estar publicada publicamente (link "pub?output=xlsx").
- 🟢 **Recomendação**: caso o link da planilha mude, atualizar apenas a constante `URL_PLANILHA_FIXA`.

---

## 👤 Autoria / Contexto

Projeto mantido por **Andres Victor** — CX, Dados & Integrações CRM, Unicesumar.
Uso interno para acompanhamento de disparos via **SocketLabs**.
