---
name: pr-review
description: Schedula un check automatico 4 volte al giorno sulle PR attive e notifica l'utente. Quando invocato manualmente, mostra l'elenco delle PR aperte e chiede su quale eseguire la review.
---

# PR Review

Questo skill ha due modalità: **check automatico** (cron) e **review manuale** (on demand).

> **Prerequisito:** Leggi dalla memoria i valori `ADO_ORG`, `ADO_PROJECT` e `ADO_REPOSITORY` raccolti durante l'Init. Se non sono presenti, esegui l'Init come descritto in CLAUDE.md.

---

## Quando lo skill viene invocato dall'utente

1. Schedula il cron job di notifica (vedi sezione **Cron Job**).
2. Esegui subito la **Modalità A — Elenco PR** per mostrare le PR attive correnti.

---

## Cron Job — Check automatico 4 volte al giorno

Crea **4 cron job** con `CronCreate`:

| Job | `cron` | Orario |
|-----|--------|--------|
| Mattina 1 | `"0 9 * * *"` | 09:00 |
| Mattina 2 | `"30 10 * * *"` | 10:30 |
| Pomeriggio 1 | `"0 14 * * *"` | 14:00 |
| Pomeriggio 2 | `"30 15 * * *"` | 15:30 |

Parametri comuni per ogni cron:
- `recurring`: `true`
- `prompt`: (vedi sotto, con i valori reali sostituiti dalla memoria)

### Prompt per ogni tick del cron

```
Controlla se ci sono PR aperte su Azure DevOps per il repository {ADO_REPOSITORY} nel progetto {ADO_PROJECT}.

Usa il tool `mcp__azure-devops__repo_list_pull_requests_by_repo_or_project` con:
- repositoryId: "{ADO_REPOSITORY}"
- project: "{ADO_PROJECT}"
- status: "active"

Se ci sono PR aperte, notifica l'utente con un messaggio del tipo:
"🔔 Ci sono <N> PR aperte in attesa di review su {ADO_REPOSITORY}: <elenco titoli con ID e autore>"

Se non ci sono PR aperte, non fare nulla e non notificare.
```

---

## Modalità A — Elenco PR (primo step della review manuale)

1. Lista tutte le PR aperte con `mcp__azure-devops__repo_list_pull_requests_by_repo_or_project`:
   - `repositoryId`: `{ADO_REPOSITORY}`
   - `project`: `{ADO_PROJECT}`
   - `status`: `"active"`

2. Mostra l'elenco all'utente in formato tabella:

   | # | ID | Titolo | Autore | Branch |
   |---|----|--------|--------|--------|
   | 1 | ... | ... | ... | ... |

3. **Chiedi all'utente su quale PR vuole eseguire la review** prima di procedere.

4. Se non ci sono PR attive, comunicarlo all'utente e terminare.

---

## Modalità B — Review su PR scelta

Una volta che l'utente ha scelto la PR, esegui la review:

1. Recupera i dettagli della PR con `mcp__azure-devops__repo_get_pull_request_by_id`:
   - `pullRequestId`: `<PR_ID>`
   - `repositoryId`: `{ADO_REPOSITORY}`
   - `project`: `{ADO_PROJECT}`

2. Analizza le modifiche dal punto di vista di:
   - Qualità del codice (leggibilità, naming, duplicazioni)
   - Potenziali bug o regressioni
   - Performance
   - Rispetto delle convenzioni del progetto
   - Sicurezza

3. Posta un commento sulla PR con `mcp__azure-devops__repo_create_pull_request_thread`:
   - `pullRequestId`: `<PR_ID>`
   - `repositoryId`: `{ADO_REPOSITORY}`
   - `project`: `{ADO_PROJECT}`
   - `content`: vedi struttura sotto

   ```
   ## 🤖 PR Review

   ### Sintesi
   <breve descrizione di cosa fa la PR>

   ### ✅ Punti positivi
   <lista>

   ### ⚠️ Suggerimenti / Problemi
   <lista con file:riga se possibile>

   ### 📋 Note finali
   <eventuali osservazioni generali>

   ---
   *Review generata il <data> alle <ora>*
   ```

4. Dopo aver postato, chiedi all'utente se vuole fare la review di un'altra PR dall'elenco.

---

## Note

- I valori `{ADO_ORG}`, `{ADO_PROJECT}`, `{ADO_REPOSITORY}` sono letti dalla memoria (raccolti durante l'Init). Sostituiscili con i valori reali nel prompt del cron prima di schedularlo.
- I cron job sono session-only: scadono dopo 3 giorni. Ricordare all'utente di reinvocare lo skill se necessario.
