# Project: Claude Azure DevOps

## Init

**OBBLIGATORIO — Da eseguire all'inizio di ogni nuova sessione di lavoro.**

Prima di qualsiasi attività, controlla se la configurazione Azure DevOps è già salvata in memoria. Se non è presente, chiedi all'utente le seguenti informazioni.

### Valori da salvare in memoria (persistenti, non sensibili)

Raccogli questi valori e salvali in memoria come tipo `project`:

| Variabile         | Descrizione                                                   | Esempio                                                                             |
| ----------------- | ------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `ADO_ORG`         | Nome dell'organizzazione Azure DevOps                         | `myorg`                                                                             |
| `ADO_PROJECT`     | Nome del progetto ADO — container di board, work items e team | `your_project`                                                               |
| `ADO_REPOSITORY`  | Nome del repository git corrente su cui si lavora             | `your_repository`                                                                  |
| `ADO_TEAM`        | Nome del team                                                 | `My Team`                                                                           |
| `ADO_PIPELINE_ID` | ID numerico della pipeline CI/CD                              | `1234`                                                                              |
| `ADO_MAIN_BRANCH` | Branch principale                                             | `master`                                                                            |
| `ADO_BOARD_URL`   | URL diretto della board del team (fornito dall'utente)        | `https://dev.azure.com/myorg/your_project/_boards/board/t/My%20Team/Stories` |

> **Nota:** `ADO_PROJECT` e `ADO_REPOSITORY` sono concetti distinti. Il progetto è il container ADO (board, work items, iterazioni). Il repository è il repo git su cui si sta lavorando — può cambiare da sessione a sessione.

### Valori da configurare in `.mcp.json` (sensibili, mai in memoria)

Verificare se il file `.mcp.json` esiste nella root del progetto. Se non esiste, crearlo. Il file deve avere questa struttura:

```json
{
  "mcpServers": {
    "azure-devops": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@azure-devops/mcp",
        "ADO_PROJECT",
        "--authentication",
        "env"
      ],
      "env": {
        "AZURE_DEVOPS_EXT_PAT": "AZURE_DEVOPS_EXT_PAT",
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

Dove:

- `ADO_PROJECT` → valore raccolto dall'utente (es. `my-project`)
- `AZURE_DEVOPS_EXT_PAT` → Personal Access Token fornito dall'utente (**segreto, mai salvare in memoria**)

Dopo aver raccolto i valori, usa queste variabili in tutti i comandi az CLI, URL e istruzioni del progetto.

## Azure DevOps

I valori di configurazione vengono raccolti durante l'**Init**. Usa sempre i valori salvati in memoria al posto dei placeholder `{ADO_*}`.

### URL utili

- **Board:** `{ADO_BOARD_URL}` _(link diretto salvato in memoria)_
- **Pull Requests:** `https://dev.azure.com/{ADO_ORG}/{ADO_PROJECT}/_git/{ADO_REPOSITORY}/pullrequests?_a=mine`
- **Pipeline:** `https://dev.azure.com/{ADO_ORG}/{ADO_PROJECT}/_build?definitionId={ADO_PIPELINE_ID}`

## Login

In caso di problemi di autenticazione, usare `AZURE_DEVOPS_EXT_PAT` nel file `.mcp.json`.

### Instructions

- When searching for work items, boards, or tasks: use project `{ADO_PROJECT}`, team `{ADO_TEAM}` — the board URL is `{ADO_BOARD_URL}`
- When searching for PRs or performing git operations: use repository `{ADO_REPOSITORY}`
- When searching for pipelines: use `definitionId={ADO_PIPELINE_ID}`
- Main branch is `{ADO_MAIN_BRANCH}`
- `ADO_PROJECT` ≠ `ADO_REPOSITORY`: always use the right scope — project for work items/board, repository for code/PRs
