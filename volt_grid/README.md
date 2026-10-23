# VoltGrid – REST API Ingestion with Azure Data Factory

An Azure Data Factory (ADF) project that ingests data from the VoltGrid REST API into Azure Data Lake Storage Gen2 (ADLS) as raw JSON. It supports paginated extraction, full and incremental loads, and per-entity watermarks.

- **Data factory:** `voltgridadf`
- **Secrets:** Azure Key Vault `voltgridkeyvault` (accessed via managed identity)
- **Sink:** ADLS Gen2 (raw container)

---

## Architecture

```
Schedule_Based_Trigger
        │
        ▼
VoltGrid Pipline (parent)
  ├─ Get userName / password from Key Vault (managed identity)
  ├─ POST /api/auth/login/  →  authToken (variable)
  └─ ForEach entity (customers, payment, sessions, stations, vehicles)
        │
        ▼
VoltGrid Each Entity Sub Pipline (child, one endpoint per run)
  ├─ Lookup  act_get_entity_watermark   (controls/<entity>.json → lastProcessed)
  ├─ Until (done = true)
  │     └─ Copy: GET /api/db/<entity>/?page=N  →  ADLS
  ├─ Page-cap check
  └─ Save new watermark (run_start captured at pipeline start)
```

### Pipelines

| Pipeline | Role |
|---|---|
| `VoltGrid Pipline` | Parent. Authenticates, then loops over entities and calls the child pipeline |
| `VoltGrid Pipline_copy1` | Copy of the parent used during development |
| `VoltGrid Each Entity Sub Pipline` | Child. Extracts all pages for a single endpoint |

### Datasets

| Dataset | Type | Notes |
|---|---|---|
| `ls_voltgrid_api` | REST | Source API |
| `RawContainer` | ADLS Gen2 | Sink. Dataset properties: `entity`, `file_name` |

---

## Source API

- **Base path:** `/api/db/<entity>/?page=N` (Vercel-hosted)
- **Auth:** `POST /api/auth/login/` returns a token. Send it as `Authorization: Token <token>` (a `Bearer` prefix returns 401)
- **Response shape:**
  ```json
  {
    "data": [ ... ],
    "pagination": { "page": 1, "page_size": 20, "total": 200007, "total_pages": 10001 }
  }
  ```
- **Incremental filter:** `updated_after=<UTC timestamp>`. Records carry `id`, `<entity>_id`, and `updated_at` (UTC, `Z` suffix). When nothing is newer, the API returns 200 with `total = 0` and `total_pages = 1`
- **Entities:** customers, payment, sessions, stations, vehicles

---

## Pipeline Parameters and Variables

**Child pipeline parameters**

| Name | Default | Purpose |
|---|---|---|
| `entity_name` | – | Endpoint to extract |
| `ingestion_date` | – | Used in the sink folder name |
| `load_type` | `full` | `full` or `incremental` |
| `watermark` | *(empty)* | Last processed timestamp for incremental loads |

**Child pipeline variables**

| Name | Default | Purpose |
|---|---|---|
| `page` | `1` | Current page number |
| `nextPage` | – | Temp variable for incrementing `page` |
| `done` | `false` | Exit flag for the Until loop |

---

## Load Strategy

- **Full load:** extracts all pages for an entity.
- **Incremental load:** passes `updated_after=<watermark>` to the API.
- **Watermark storage:** one control file per entity at `controls/<entity>.json`, with key `lastProcessed`.
  - Read by the Lookup `act_get_entity_watermark` before the Until loop.
  - Written after the Until loop and the page-cap check, using `run_start` captured at the start of the run (so records updated mid-run are not missed).

### Output layout

```
<raw container>/<entity>/ingestion_date=YYYY-MM-DD/page_N.json
```

---

## Development Progress

| Stage | Scope |
|---|---|
| Day 2 | Payments only, 1 page, full load, fixed `payments.json` |
| Day 3 | All pages via Until, full + incremental (`updated_after`), partitioned sink, all endpoints via ForEach, watermark handling |

---

## Known Issues

1. **Past-the-end pagination returns 404.** The API responds with `404 {"detail":"Invalid page."}` when `page` exceeds `total_pages`, so the Copy activity fails at the end of pagination.
2. **Until page cap too low for large entities.** The cap is 10 pages, but `customers` has 10,001 pages at page size 20 (200,007 rows). The run fails at `act_fail_load` when the cap is hit. The current Until-loop design is being kept for now.
3. **ForEach concurrency.** Unchecking *Sequential* on the parent ForEach produced errors. The ForEach currently contains two Execute Pipeline activities (`act_execute_sub_pipline` and a copy1).

---

## Setup

1. Create Key Vault `voltgridkeyvault` with secrets for the API `userName` and `password`.
2. Grant the ADF managed identity *Get/List* permission on those secrets.
3. Grant the ADF managed identity *Storage Blob Data Contributor* on the ADLS account.
4. Create the datasets `ls_voltgrid_api` and `RawContainer`.
5. Publish the pipelines and attach `Schedule_Based_Trigger` to the parent pipeline.

---

## Possible Next Steps

- Stop pagination using `total_pages` from the first response instead of a fixed page cap.
- Handle the end-of-pages 404 gracefully.
- Re-enable parallel ForEach once the concurrency errors are understood.
- Add a `pipeline_audit` Delta table for run logging.
