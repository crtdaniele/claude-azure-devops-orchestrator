---
name: ado-workflow
description: Azure DevOps development workflow — gestisce il ciclo di vita di un work item durante lo sviluppo: attivazione, aggiornamento descrizione, e collegamento alla PR. Da usare ogni volta che si inizia uno sviluppo o si apre una PR su questo progetto.
---

# ADO Workflow

Questo skill definisce il workflow obbligatorio da seguire quando si sviluppa su questo progetto con Azure DevOps.

> **Prerequisito:** Prima di eseguire qualsiasi comando, leggi dalla memoria i seguenti valori raccolti durante l'Init. Se non sono presenti, esegui l'Init come descritto in CLAUDE.md.
>
> | Variabile         | Usata per                                                 |
> | ----------------- | --------------------------------------------------------- |
> | `ADO_ORG`         | organizzazione Azure DevOps                               |
> | `ADO_PROJECT`     | work items, board, iterazioni — es. `am-digital-products` |
> | `ADO_REPOSITORY`  | operazioni git e PR — es. `be-app-segafredo`              |
> | `ADO_TEAM`        | team della board — es. `Burst Devs Team`                  |
> | `ADO_BOARD_URL`   | link diretto alla board del team                          |
> | `ADO_MAIN_BRANCH` | branch target delle PR                                    |
>
> **Importante:** `ADO_PROJECT` e `ADO_REPOSITORY` sono distinti. Tutte le operazioni su work item usano `ADO_PROJECT`; tutte le operazioni su PR e branch usano `ADO_REPOSITORY`.

## Quando usare questo skill

- Quando si inizia a lavorare su un task / feature / fix
- Quando si apre una Pull Request

---

## Step 1 — Inizio sviluppo (prima di scrivere codice)

1. **Identificare il work item**: chiedere all'utente l'ID del work item se non è già specificato nel contesto. I work item si trovano sulla board `{ADO_BOARD_URL}` nel progetto `{ADO_PROJECT}`.

   Se è necessario elencare i work item della board, usare **sempre** il tool `mcp__azure-devops__wit_list_backlog_work_items` scoped al team configurato — **mai** `wit_my_work_items` (che restituisce item da tutti i team e progetti):

   ```
   mcp__azure-devops__wit_list_backlog_work_items:
     project: {ADO_PROJECT}
     team:    {ADO_TEAM}
     backlogId: Microsoft.RequirementCategory   # User Stories
   ```

   Per ottenere anche i dettagli (titolo, stato) degli ID restituiti, usare `wit_get_work_items_batch_by_ids` con i campi `System.Id`, `System.Title`, `System.State`, `System.WorkItemType`.

   > **Perché:** `wit_my_work_items` aggrega item da tutte le board del progetto, mescolando lavori di team diversi. Usare il backlog del team garantisce di operare esclusivamente sulla board `{ADO_BOARD_URL}`.

2. **Mettere il work item in stato "Active"**:

Usa il tool MCP `mcp_ado_wit_update_work_item` con i parametri:

- `workItemId`: `<WORK_ITEM_ID>`
- `fields`: `{ "System.State": "Active" }`

3. **Aggiornare la descrizione del work item** con un piano ad alto livello di implementazione:

Usa il tool MCP `mcp_ado_wit_update_work_item` con i parametri:

- `workItemId`: `<WORK_ITEM_ID>`
- `fields`: `{ "System.Description": "<HTML con piano implementativo>" }`

La descrizione deve includere:

- Obiettivo dell'implementazione
- Componenti / file principali che verranno modificati
- Approccio tecnico scelto

La descrizione deve includere:

- Obiettivo dell'implementazione
- Componenti / file principali che verranno modificati
- Approccio tecnico scelto

---

## Step 2 — Apertura Pull Request

### Naming convention del branch

Il branch deve seguire questo formato:

```
<tipo>/<anno>/<id-task>
```

- `<tipo>`: scegliere in base alla natura del lavoro — `feature`, `fix`, `refactor`, `chore`, `hotfix`, `docs`
- `<anno>`: anno corrente (es. `2026`)
- `<id-task>`: ID del work item Azure DevOps (es. `1234`)

Esempi: `feature/2026/1234`, `fix/2026/5678`, `refactor/2026/9012`

Se il branch non esiste ancora, crearlo prima di aprire la PR:

```bash
git checkout -b <tipo>/<anno>/<id-task>
git push -u origin <tipo>/<anno>/<id-task>
```

### Creazione PR

Usa il tool MCP `mcp_ado_repo_create_pull_request` con i parametri:

- `repositoryId`: `{ADO_REPOSITORY}`
- `sourceRefName`: `refs/heads/<tipo>/<anno>/<id-task>`
- `targetRefName`: `refs/heads/{ADO_MAIN_BRANCH}`
- `title`: `<TITOLO>`
- `description`: `<DESCRIZIONE>\n\nAB#<WORK_ITEM_ID>`
- `workItemRefs`: `[{ "id": "<WORK_ITEM_ID>" }]`

Poi collega il work item alla PR con `mcp_ado_wit_link_work_item_to_pull_request`:

- `workItemId`: `<WORK_ITEM_ID>`
- `pullRequestId`: `<PR_ID>` (restituito dal tool precedente)
- `repositoryId`: `{ADO_REPOSITORY}`

---

## Note

- I valori `{ADO_ORG}`, `{ADO_PROJECT}`, `{ADO_REPOSITORY}`, `{ADO_TEAM}`, `{ADO_BOARD_URL}` e `{ADO_MAIN_BRANCH}` sono letti dalla memoria (raccolti durante l'Init).
- Non serve passare `--org` perché l'MCP è già configurato con l'organizzazione nel `.mcp.json`.
- **Work items e board** → sempre su `{ADO_PROJECT}` + `{ADO_TEAM}` per restare scoped alla board corretta
- **PR e operazioni git** → sempre su `{ADO_REPOSITORY}` (es. `be-app-segafredo`)
- **Mai usare `wit_my_work_items`** per cercare task — restituisce item da tutti i team del progetto
- Se l'utente cambia repository, aggiornare `ADO_REPOSITORY` in memoria prima di procedere.
