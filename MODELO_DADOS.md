# 📑 Modelo de Dados — Estrutura da Planilha

O dashboard espera uma planilha Google Sheets publicada (`pub?output=xlsx`) com **3 abas lógicas**. O código tenta localizar essas abas por nome, com tolerância a variações (maiúsculas/minúsculas e acentuação), através de uma função auxiliar `getSheet(names)`.

> 🟡 **Hipótese**: como a função `getSheet` foi vista parcialmente (linhas 694–717 truncadas na leitura), a lista exata de aliases aceitos por aba não pôde ser 100% confirmada neste documento. 🟢 **Recomendação**: validar diretamente no código-fonte (`index.html`, função `fetchOnlineData`) os nomes alternativos aceitos antes de renomear abas na planilha.

---

## Aba `Geral`

Usada como fonte de:
- Última atualização (`Config`)
- KPIs de Realizado, Projetado e Uso Geral

| Coluna | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `Mes` | texto (`Jan`...`Dez`) | ✅ | Mês de referência da linha |
| `Realizado` | número | ✅ | Valor total disparado no mês (CRM Alunos) |
| `Projetado` | número | ⚪ | Valor projetado/planejado para o mês (usado na soma anual; o projetado mensal exibido é `Realizado + Agendados`) |
| `UsoGeral` | número | ✅ | Valor total usado do pacote compartilhado (Alunos + Mercado) no mês |
| `Última atualização` | data (serial Excel, `dd/mm` ou `dd/mm/aaaa`) | ✅ | Data/hora da última sincronização — usada para o filtro automático de mês e cálculo do "ritmo diário" |

> 🔵 **Fato observado**: a aba `Geral` é usada tanto como `data.Config` quanto `data.Geral` (`rawData = { Config: geralRows, Geral: geralRows, ... }`), ou seja, **é a mesma aba para ambos os papéis**.

---

## Aba `Historico`

Usada para o gráfico de **Sazonalidade Mensal** (comparação ano a ano).

| Coluna | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `Mes` | texto (`Jan`...`Dez`) | ✅ | Mês de referência |
| `v25` | número | ✅ | Valor disparado em **2025** (referência) |
| `v26` | número | ✅ | Valor disparado em **2026** (atual) |

Escala do gráfico: cada barra é calculada como `valor / 15.000.000 * 100` (altura em %).

---

## Aba `Campanhas`

Usada para as 4 tabelas de eventos: **Agendados, Enviados, Pausados, Cancelados**.

| Coluna | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `Dia` | data (serial Excel ou `dd/mm[/aaaa]`) | ✅ | Data do disparo/evento |
| `Base` | número | ✅ | Tamanho da base/lista (usado para somatórios e cálculo de "Peso") |
| `Status` | texto | ✅ | Determina a tabela de destino — ver regra abaixo |
| `Chamado` | texto | ⚪ | Número/ID do chamado relacionado |
| `Tema` | texto | ⚪ | Assunto/tema da campanha (exibido na coluna "Tema" das tabelas Agendados/Cancelados) |
| `MesRef` | texto (`Jan`...`Dez`) | ⚪ | Mês de referência alternativo para filtro (usado quando `Dia` não corresponde ao mês desejado) |

### Regra de classificação por `Status`

O código usa `status.toLowerCase().includes(...)` — portanto **qualquer texto que contenha** os termos abaixo é aceito:

| Termo contido em `Status` | Tabela destino | ID da tabela | Soma exibida em |
|---|---|---|---|
| `"agendado"` | Agendados | `#table-agendados` | `#sum-agendados` |
| `"envia"` (ex: "Enviado", "Enviando") | Enviados | `#table-pausados` ⚠️ | `#sum-pausados` ⚠️ |
| `"pausa"` (ex: "Pausado") | Pausados | `#table-pausados-status` | `#sum-pausados-status` |
| `"cancela"` (ex: "Cancelado") | Cancelados | `#table-cancelados` | `#sum-cancelados` |

> 🔵 **Fato observado**: há uma inconsistência de nomenclatura nos IDs do HTML — a tabela de **"Enviados"** usa o ID `#table-pausados` / `#sum-pausados`, enquanto a tabela de **"Pausados"** usa `#table-pausados-status` / `#sum-pausados-status`. Isso é apenas uma questão de nomenclatura de IDs (legado), **a lógica de classificação funciona corretamente**, mas pode causar confusão em manutenções futuras.
> 🟢 **Recomendação**: caso o projeto evolua, renomear os IDs para `#table-enviados` / `#sum-enviados` para evitar erros de manutenção.

### Filtro por mês (quando `selectedMonth` está ativo)

Uma campanha é exibida se **qualquer** uma das condições for verdadeira:
- `MesRef === selectedMonth`, **ou**
- O mês extraído de `Dia` (via `excelValueToDate`) corresponde a `selectedMonth`

### Cálculo do "Peso" (coluna exibida nas tabelas)

```
Peso (%) = (Base / META_ALUNOS) * 100
```
Exibido como badge azul (`weight-badge`) em todas as tabelas.

---

## Conversão de Datas (`excelValueToDate`)

Função central para interpretar datas vindas da planilha, aceitando 3 formatos:

| Formato de entrada | Tratamento |
|---|---|
| `Date` (objeto JS) | Retornado diretamente |
| `number` (serial Excel/Sheets) | Convertido a partir da data-base `30/12/1899`, somando dias inteiros + fração de horas/minutos |
| `string` `"dd/mm"` ou `"dd/mm/aaaa"` | Convertido para `Date`; se o ano não for informado, usa o ano atual |
| outro/vazio | Retorna `null` |

> ⚠️ **Atenção**: se a coluna `Dia`/`Última atualização` estiver formatada como **texto livre** em um formato diferente de `dd/mm` ou `dd/mm/aaaa`, a conversão falhará silenciosamente (retorna `null`), afetando o filtro automático de mês e o cálculo de ritmo diário.

---

## Cálculo de KPIs — Resumo das Fórmulas

| KPI | Fórmula | Onde aparece |
|---|---|---|
| Realizado | `Geral.Realizado` (do mês ou soma anual) | `#kpi-realizado` |
| % da Meta | `Realizado / metaEmail * 100` | `#kpi-pct-meta` |
| Projetado Total | `Realizado + Σ(Base agendados do período)` | `#kpi-projetado` |
| % Projetado da Meta | `Projetado / metaEmail * 100` | `#kpi-projetado-pct` |
| Uso Alunos (%) | `Realizado / metaPacote * 100` | `#kpi-uso-alunos` |
| Uso Geral (valor) | `Geral.UsoGeral` (do mês ou soma anual) | `#kpi-uso-geral` |
| Uso Geral (%) | `UsoGeral / metaPacote * 100` | `#kpi-uso-geral-pct` |
| Saldo Disponível | `100 - %daMeta` | `#kpi-saldo` |

Onde:
- `metaEmail = META_ALUNOS` (visão mensal) ou `META_ALUNOS * 12` (visão anual)
- `metaPacote = CAPACIDADE_MAX` (visão mensal) ou `CAPACIDADE_MAX * 12` (visão anual)

---

## Status Diário da Meta (`calcularStatusMetaTemporal`)

Compara o **ritmo ideal** vs. o **ritmo real** de envio, apenas quando há mês selecionado:

```
percentualIdeal = (dia da atualização / dias no mês) * 100
percentualAtual = (Realizado / META_ALUNOS) * 100
diferenca       = percentualIdeal - percentualAtual
dentroDaMeta    = diferenca >= 0
```

- Se `dentroDaMeta = true` → status **"Dentro da meta"** (verde)
- Se `dentroDaMeta = false` → status **"Acima da meta"** (vermelho)
- Se o mês selecionado **for diferente** do mês da `Última atualização` → exibe aviso âmbar informando que o comparativo diário só está disponível para o mês da última atualização
- Sem mês selecionado → exibe "Cálculo anual" (cinza)
