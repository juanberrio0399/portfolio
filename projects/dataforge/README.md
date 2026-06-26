# DataForge — Technical overview

ETL pipeline + analytical layer that reconciles an international courier's customs operation.
Turns business-generated Excel files into columnar Parquet with incremental dedup, and exposes
the result as DuckDB views consumed from BI tools, notebooks and a serverless cloud dashboard.

> Production system. Client code is private; this page documents the architecture with anonymized details.

## Architecture

```
Excel (SharePoint)
      │  readers.py     parallel scan + read (calamine/openpyxl)
      ▼
  DataFrame
      │  transforms.py  column cleaning, typing, join-key generation
      ▼
  clean DataFrame
      │  writers.py     merge + incremental dedup → single consolidated Parquet
      ▼
  data/processed/<type>/data_YYYYMMDD_HHMMSS.parquet
      │  warehouse.py   CREATE OR REPLACE VIEW per type
      ▼
  courierbox.duckdb  →  Cloudflare R2  →  React + DuckDB-WASM (Cloudflare Pages)
```

The incremental catalog (`catalog.json`) stores each Excel's modification date, so only what
changed gets reprocessed.

## Data model

5 source tables consolidated and joined through declarative relationships:

| Table                | Key          |
|----------------------|--------------|
| `settlements`        | `key_mawb`   |
| `settlements_temu`   | `key_mawb`   |
| `manifests`          | `key_mawb`   |
| `payments`           | `package_no` |
| `authority_690`      | `package_no` |

The **manifest** is the bridge that carries both identifier systems, enabling the
package-level (1:1) tax reconciliation chain — see the
[case study](../../case-studies/01-dataforge-customs-reconciliation_EN.md).

## Highlights

- ~4,100 lines of Python in a layered architecture (readers → transforms → writers)
- Incremental ingestion + dedup; Parquet columnar storage; DuckDB view warehouse
- Serverless cloud delivery: Cloudflare R2 + Pages, React + DuckDB-WASM, no backend
- Unattended automation 3×/day (Windows Task Scheduler + GitHub Actions)
- Production hardening: UTF-8 under cron, per-run logging, failure isolation

## Stack

`Python` · `Pandas` · `DuckDB` · `Parquet` · `Cloudflare R2/Pages` · `React + DuckDB-WASM` · `GitHub Actions` · `Power Automate`
