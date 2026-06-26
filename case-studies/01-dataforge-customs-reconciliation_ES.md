# DataForge — Conciliación tributaria aduanera, guía a guía

> **Stack:** Python · DuckDB · Parquet · Cloudflare R2/Pages · React + DuckDB-WASM · GitHub Actions · Power Automate
> **Rol:** Diseño, desarrollo y operación end-to-end (1 persona)
> **Estado:** En producción — corre 3 veces al día sin supervisión
>
> *Case study basado en un sistema real. Nombres de cliente y datos anonimizados.*

## Problema

Una operación de courier internacional liquida miles de guías aduaneras al mes. Cada guía debe pagar un impuesto aduanero a la autoridad tributaria y, en paralelo, el marketplace de e-commerce debe reembolsar esos impuestos. El problema: **nadie sabía, guía por guía, qué estaba pagado y qué estaba pendiente de reembolso.**

La razón es técnica: la liquidación identifica cada guía con un número (`guía_hija`), pero el pago a la autoridad y el reembolso del marketplace la identifican con un código distinto. **Los dos mundos no tenían una llave común.** El control se hacía a mano, por master, en Excel — lento, propenso a error y sin visibilidad real al nivel que importa: la guía individual.

## Contexto

- **Fuentes:** 5 archivos Excel distintos generados por el negocio (liquidaciones, manifiestos, pagos del marketplace, consolidado de pagos a la autoridad), actualizados continuamente en SharePoint.
- **Volumen:** millones de registros acumulados; miles de guías nuevas por corrida.
- **Restricciones:** presupuesto cero para infraestructura, debía correr en el PC de la operación sin afectar el trabajo diario, y el resultado tenía que ser consultable por dirección sin instalar nada.

## Solución

Un pipeline ETL que convierte los Excel del negocio en un **warehouse analítico consultable** y publica un **dashboard cloud serverless** que responde, por guía, las dos preguntas que importan: *¿está pagada a la autoridad?* y *¿ya la reembolsó el marketplace?*

La clave del modelado: **el manifiesto es el único documento que contiene ambos identificadores**, así que se usa como puente para encadenar la liquidación con los pagos:

```
liquidaciones.guía_hija → manifiestos → package_no
        → consolidado_impuesto ⇒  pagado a la autoridad
        → pagos_marketplace    ⇒  reembolsado
```

## Arquitectura

```
  Excel del negocio (SharePoint)
        │
        │  readers.py    escaneo + lectura en paralelo (calamine/openpyxl)
        ▼
   DataFrame crudo
        │
        │  transforms.py limpieza, tipado, generación de llaves de join
        ▼
   DataFrame limpio
        │
        │  writers.py    merge + dedup incremental → Parquet consolidado
        ▼
  data/processed/<tipo>/data_YYYYMMDD.parquet
        │
        │  warehouse.py  CREATE OR REPLACE VIEW por tipo (DuckDB)
        ▼
  warehouse.duckdb   ──run_daily──►  Cloudflare R2  ──►  React + DuckDB-WASM (Pages)
        ▲                                                      (dashboard directivo)
        │
   catalog.json (tracker incremental: mtime por archivo → reprocesa solo lo que cambió)
```

**Decisiones clave**
- **DuckDB en vez de Postgres:** analítica columnar, cero servidor, archivos portables. Ideal para un solo operador y presupuesto cero.
- **Parquet como capa intermedia:** compresión columnar + dedup incremental; el warehouse se reconstruye en segundos.
- **Cloudflare R2 + DuckDB-WASM en vez de un backend:** el dashboard consulta los Parquet **directamente en el navegador**. No hay API, ni base de datos en la nube, ni servidor que mantener o pagar.
- **Configuración declarativa:** fuentes, tipos y relaciones viven en un solo archivo de config, separados de la lógica del pipeline.
- **Ingesta incremental por catálogo:** se guarda la fecha de modificación de cada Excel; en cada corrida solo se reprocesa lo que cambió.

## Stack técnico

- **Ingesta:** Python, lectura paralela de Excel (calamine/openpyxl)
- **Procesamiento:** Pandas, ~4.100 líneas en arquitectura por capas (readers → transforms → writers)
- **Almacenamiento:** Parquet (capa intermedia) + DuckDB (warehouse de vistas)
- **Presentación:** React + DuckDB-WASM en Cloudflare Pages
- **Orquestación:** Windows Task Scheduler (3x/día) + GitHub Actions; sync a Cloudflare R2

## Resultados

- **Visibilidad guía a guía** donde antes solo había control manual por master: por cada guía se sabe si está pagada a la autoridad y si fue reembolsada.
- **Conciliación automatizada de 5 fuentes** que antes se cruzaban a mano en Excel.
- **Corre 3 veces al día sin intervención**, con ingesta incremental (solo procesa lo que cambió) y logging por corrida.
- **Costo de infraestructura ≈ $0:** warehouse local + tier gratuito de Cloudflare; dashboard sin servidor que mantener.
- **Robusto en producción:** manejo de encoding UTF-8 bajo cron de Windows y aislamiento de fallos (un paso cloud que falla no tumba la corrida local).

## Lecciones aprendidas

- **El modelado de la llave puente fue el 80% del valor.** El reto no era técnico de herramientas, sino encontrar que el manifiesto era el único enlace entre los dos sistemas de identificación.
- **DuckDB + Parquet + Cloudflare es un stack subestimado** para analítica de bajo costo: capacidades de "data warehouse" sin el costo ni la operación de uno.
- **Diseñar para que corra desatendido cambia el código:** logging, idempotencia y aislamiento de fallos importan tanto como la lógica de negocio.
