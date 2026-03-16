---
name: check-pipeline
description: Verifica lo stato dell'ultima pipeline CI/CD del progetto o lancia una nuova esecuzione. Usare quando si vuole sapere se la pipeline è passata o fallita, oppure triggerare manualmente una nuova build.
---

# Check Pipeline

Questo skill permette di verificare lo stato della pipeline CI/CD e/o triggerare una nuova esecuzione.

> **Prerequisito:** Leggi dalla memoria i valori `ADO_ORG`, `ADO_PROJECT` e `ADO_PIPELINE_ID` raccolti durante l'Init. Se non sono presenti, esegui l'Init come descritto in CLAUDE.md.

## Quando usare questo skill

- Dopo aver aperto una PR, per verificare se la pipeline è passata
- Per controllare lo stato dell'ultima build
- Per triggerare manualmente una nuova pipeline

---

## Azione 1 — Verifica stato pipeline

Usa il tool MCP `mcp_ado_pipelines_list_runs` con i parametri:
- `pipelineId`: `{ADO_PIPELINE_ID}`
- `project`: `{ADO_PROJECT}`

Mostra le ultime 5 esecuzioni con stato e risultato.

### Interpretazione dello stato

| Stato | Significato |
|-------|-------------|
| `succeeded` | Pipeline completata con successo |
| `failed` | Pipeline fallita — vedere i log per i dettagli |
| `running` / `inProgress` | Pipeline in corso |
| `canceled` | Pipeline annullata manualmente |
| `partiallySucceeded` | Completata con warning |

In caso di stato `failed`, recupera i log con `mcp_ado_pipelines_get_build_log`:
- `buildId`: `<RUN_ID>`
- `project`: `{ADO_PROJECT}`

e riassumi il motivo del fallimento.

---

## Azione 2 — Lancia una nuova pipeline

Usa il tool MCP `mcp_ado_pipelines_run_pipeline` con i parametri:
- `pipelineId`: `{ADO_PIPELINE_ID}`
- `project`: `{ADO_PROJECT}`
- `branch`: `<BRANCH>` (chiedere all'utente se non specificato)

Dopo il lancio, riporta il `runId` e il link diretto all'esecuzione:
`https://dev.azure.com/{ADO_ORG}/{ADO_PROJECT}/_build/results?buildId=<RUN_ID>`

---

## Note

I valori `{ADO_ORG}`, `{ADO_PROJECT}`, `{ADO_PIPELINE_ID}` sono letti dalla memoria (raccolti durante l'Init).
