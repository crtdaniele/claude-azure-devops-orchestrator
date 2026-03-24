# Claude Azure DevOps Orchestrator

Configurazione di **Claude Code** per integrare Azure DevOps tramite MCP server. Permette a Claude di interagire direttamente con board, work items, PR e pipeline senza uscire dall'editor.

---

## Requisiti

- [Claude Code](https://claude.ai/code) installato
- Un Personal Access Token (PAT) Azure DevOps con permessi su: Work Items, Code, Build
- Node.js / npx disponibile

---

## Configurazione

### 1. Crea il file `.mcp.json` nella root del progetto

```json
{
  "mcpServers": {
    "azure-devops": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@azure-devops/mcp",
        "<ADO_PROJECT>",
        "--authentication",
        "env"
      ],
      "env": {
        "AZURE_DEVOPS_ORG_URL": "https://dev.azure.com/<ADO_ORG>/",
        "AZURE_DEVOPS_EXT_PAT": "<IL_TUO_PAT>"
      }
    },
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    }
  }
}
```

Sostituisci:
- `<ADO_ORG>` → nome dell'organizzazione Azure DevOps
- `<ADO_PROJECT>` → nome del progetto ADO (es. `am-digital-products`)
- `<IL_TUO_PAT>` → Personal Access Token (**non committare mai questo file**)

> Aggiungi `.mcp.json` al `.gitignore`.

### 2. Prima sessione — Init

All'avvio di ogni sessione Claude legge dalla memoria i valori di configurazione. Se non sono presenti, li chiede all'utente e li salva.

| Variabile | Descrizione | Esempio |
|-----------|-------------|---------|
| `ADO_ORG` | Nome organizzazione Azure DevOps | `myorg` |
| `ADO_PROJECT` | Progetto ADO — contiene board, work items, team | `your_project` |
| `ADO_REPOSITORY` | Repository git corrente su cui si lavora | `your_repository` |
| `ADO_TEAM` | Nome del team | `My Team` |
| `ADO_PIPELINE_ID` | ID numerico della pipeline CI/CD | `1234` |
| `ADO_MAIN_BRANCH` | Branch principale | `master` |
| `ADO_BOARD_URL` | URL diretto della board del team | `https://dev.azure.com/...` |

> **`ADO_PROJECT` ≠ `ADO_REPOSITORY`** — il progetto è il container ADO (board, work items); il repository è il repo git su cui si sta sviluppando, e può cambiare da sessione a sessione.

---

## Skills disponibili

### `/ado-workflow`

Gestisce il ciclo di vita di un work item durante lo sviluppo. Da invocare all'inizio di un task e all'apertura di una PR.

**Step 1 — Inizio sviluppo:**
1. Identifica il work item sulla board (`ADO_PROJECT` + `ADO_TEAM`)
2. Porta il work item in stato **Active**
3. Aggiorna la descrizione con il piano implementativo

**Step 2 — Apertura PR:**
1. Crea il branch con naming convention `<tipo>/<anno>/<id-task>` (es. `feature/2026/1234`)
2. Apre la PR su `ADO_REPOSITORY` verso `ADO_MAIN_BRANCH`
3. Collega il work item alla PR

---

### `/check-pipeline`

Verifica lo stato della pipeline CI/CD o lancia una nuova esecuzione.

**Cosa fa:**
- Lista le ultime esecuzioni della pipeline (`ADO_PIPELINE_ID`) con stato e risultato
- In caso di fallimento, recupera i log e riassume il motivo
- Permette di triggerare manualmente una nuova build su un branch specifico

---

### `/pr-review`

Schedula una review automatica delle PR attive. Gira 4 volte al giorno e posta un commento strutturato su ogni PR non ancora revisionata.

**Cosa fa:**
- Lista le PR aperte su `ADO_REPOSITORY`
- Per ogni PR analizza: qualità del codice, potenziali bug, performance, sicurezza, convenzioni
- Posta un commento con sintesi, punti positivi e suggerimenti
- Salta le PR che hanno già ricevuto una review automatica

**Invocazione manuale:** mostra l'elenco delle PR aperte e chiede su quale eseguire la review.

> Il job è session-only e scade dopo 3 giorni — reinvocarlo se necessario.

---

## Struttura del progetto

```
.
├── .mcp.json                          # Configurazione MCP (locale, non committare)
├── CLAUDE.md                          # Istruzioni per Claude Code
└── .claude/
    └── skills/
        ├── ado-workflow/SKILL.md
        ├── check-pipeline/SKILL.md
        └── pr-review/SKILL.md
```
