# DataForge — Package-Level Customs Tax Reconciliation

> **Stack:** Python · DuckDB · Parquet · Cloudflare R2/Pages · React + DuckDB-WASM · GitHub Actions · Power Automate
> **Role:** End-to-end design, development and operation (solo)
> **Status:** In production — runs 3× a day, unattended
>
> *Case study based on a real system. Client name and data anonymized.*

## Problem

An international courier operation clears thousands of customs shipments per month. Each shipment owes a customs tax to the tax authority and, in parallel, the e-commerce marketplace must refund that tax. The problem: **nobody knew, shipment by shipment, what was paid and what was still pending refund.**

The root cause is technical: the settlement file identifies each shipment with one number (`child_waybill`), but the authority payment and the marketplace refund identify it with a different code. **The two worlds had no common key.** Control was done manually, per master waybill, in Excel — slow, error-prone, and with no real visibility at the level that matters: the individual shipment.

## Context

- **Sources:** 5 different business-generated Excel files (settlements, manifests, marketplace payments, authority-payment consolidation), continuously updated in SharePoint.
- **Volume:** 25M+ accumulated records (growing ~2M/month); thousands of new shipments per run.
- **Constraints:** zero infrastructure budget, had to run on the operation's PC without disrupting daily work, and the result had to be queryable by leadership with nothing to install.

## Solution

An ETL pipeline that turns the business's Excel files into a **queryable analytical warehouse** and ships a **serverless cloud dashboard** that answers, per shipment, the two questions that matter: *is it paid to the authority?* and *has the marketplace refunded it?*

The modeling insight: **the manifest is the only document that contains both identifiers**, so it's used as a bridge to chain the settlement to the payments:

```
settlements.child_waybill → manifests → package_no
        → authority_tax_consolidation ⇒  paid to authority
        → marketplace_payments        ⇒  refunded
```

## Architecture

```
  Business Excel files (SharePoint)
        │
        │  readers.py    parallel scan + read (calamine/openpyxl)
        ▼
   Raw DataFrame
        │
        │  transforms.py cleaning, typing, join-key generation
        ▼
   Clean DataFrame
        │
        │  writers.py    merge + incremental dedup → consolidated Parquet
        ▼
  data/processed/<type>/data_YYYYMMDD.parquet
        │
        │  warehouse.py  CREATE OR REPLACE VIEW per type (DuckDB)
        ▼
  warehouse.duckdb   ──run_daily──►  Cloudflare R2  ──►  React + DuckDB-WASM (Pages)
        ▲                                                    (leadership dashboard)
        │
   catalog.json (incremental tracker: mtime per file → reprocess only what changed)
```

**Key decisions**
- **DuckDB over Postgres:** columnar analytics, zero server, portable files. Ideal for a single operator on a zero budget.
- **Parquet as the intermediate layer:** columnar compression + incremental dedup; the warehouse rebuilds in seconds.
- **Cloudflare R2 + DuckDB-WASM instead of a backend:** the dashboard queries the Parquet files **directly in the browser**. No API, no cloud database, no server to maintain or pay for.
- **Declarative configuration:** sources, types and relationships live in a single config file, separate from pipeline logic.
- **Catalog-based incremental ingestion:** each Excel's modification date is stored; every run only reprocesses what changed.

## Tech stack

- **Ingestion:** Python, parallel Excel reading (calamine/openpyxl)
- **Processing:** Pandas, ~4,100 lines in a layered architecture (readers → transforms → writers)
- **Storage:** Parquet (intermediate) + DuckDB (view warehouse)
- **Presentation:** React + DuckDB-WASM on Cloudflare Pages
- **Orchestration:** Windows Task Scheduler (3×/day) + GitHub Actions; sync to Cloudflare R2

## Results

- **Shipment-level visibility** where there was only manual per-master control: for every shipment you know whether it's paid to the authority and whether it's been refunded.
- **Automated reconciliation of 5 sources** that were previously cross-checked by hand in Excel.
- **Runs 3× a day unattended**, with incremental ingestion (only processes what changed) and per-run logging.
- **Infrastructure cost ≈ $0:** local warehouse + Cloudflare free tier; dashboard with no server to maintain.
- **Production-hardened:** UTF-8 encoding handling under Windows cron and failure isolation (a failing cloud step doesn't bring down the local run).

## Lessons learned

- **Modeling the bridge key was 80% of the value.** The challenge wasn't tooling — it was discovering that the manifest was the only link between the two identifier systems.
- **DuckDB + Parquet + Cloudflare is an underrated low-cost analytics stack:** data-warehouse capabilities without the cost or the operations of one.
- **Designing for unattended runs changes the code:** logging, idempotency and failure isolation matter as much as the business logic.
