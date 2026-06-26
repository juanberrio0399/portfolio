<h1 align="center">Juan Berrio — Portfolio</h1>
<h3 align="center">Cloud &amp; Data Engineer · building toward Cloud Architect ☁️</h3>

<p align="center">
Serverless data platforms, ETL pipelines and cloud dashboards that turn messy business data into reliable, low-cost decision systems.
</p>

<p align="center">
  <a href="https://juanberrio0399.github.io"><img src="https://img.shields.io/badge/🌐%20Live%20Portfolio-juanberrio0399.github.io-22d3ee?style=for-the-badge"/></a>
  <a href="https://www.linkedin.com/in/juanberrioo"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white"/></a>
  <a href="mailto:juandyb99@gmail.com"><img src="https://img.shields.io/badge/Email-EA4335?style=for-the-badge&logo=gmail&logoColor=white"/></a>
  <img src="https://img.shields.io/badge/Remote-GMT--5-2ea44f?style=for-the-badge"/>
</p>

---

## 👋 About / Sobre mí

**EN** — Cloud & Data Engineer with 6+ years turning operational and financial data into systems that run themselves. I think in architecture: separating compute, storage and presentation, choosing serverless over servers, and optimizing for cost (some systems run on ~$0 infrastructure). Currently studying **AWS** and **Systems Engineering** on the path to **Cloud Solutions Architect**.

**ES** — Ingeniero Cloud & Datos con 6+ años convirtiendo datos operativos y financieros en sistemas que se operan solos. Diseño con mentalidad de arquitectura: separo cómputo, almacenamiento y presentación, prefiero serverless y optimizo costos (algunos sistemas corren con ~$0 de infraestructura). Estudiando **AWS** e **Ingeniería de Sistemas**, en camino a **Arquitecto de Soluciones Cloud**.

---

## 🧰 Tech stack

| Área | Tecnologías |
|---|---|
| **Cloud & DevOps** | Cloudflare (R2 · Pages · Workers) · AWS *(en formación)* · GitHub Actions · Git · Serverless |
| **Data Engineering** | Python · Pandas · DuckDB · Parquet · SQL · ETL · Data Warehousing · Dimensional Modeling |
| **BI & Web** | Power BI · Power Query · React · TypeScript · DuckDB-WASM |
| **AI / Automation** | LLM integration · Prompt engineering · AI-assisted workflows · Playwright (RPA) · Power Automate |

---

## 🚀 Projects

> La mayoría de repos son privados (trabajo de cliente). Aquí están los write-ups con arquitectura y resultados; el código sensible se mantiene privado y los datos están anonimizados.

### ☁️ DataForge Cloud — Serverless analytics platform
Arquitectura cloud híbrida: un worker ETL local publica en **Cloudflare R2**, y un **dashboard serverless** (Cloudflare Pages + React + **DuckDB-WASM**) consulta Parquet *directo en el navegador* — sin backend, sin base de datos que mantener, ~$0 de infra. Un camino paralelo 100% nube (Power Automate → R2 → GitHub Actions) ingiere otras fuentes.
**Stack:** `Cloudflare R2/Pages` · `React + DuckDB-WASM` · `GitHub Actions` · `Power Automate`
📖 **[Case study (EN)](case-studies/01-dataforge-customs-reconciliation_EN.md)** · **[Case study (ES)](case-studies/01-dataforge-customs-reconciliation_ES.md)**

### 🦆 DataForge — ETL pipeline & analytical warehouse
ETL en producción (~4.100 líneas, arquitectura `readers → transforms → writers`) que ingiere Excel de forma incremental a **Parquet** y expone un warehouse **DuckDB**. Conciliación tributaria guía-a-guía cruzando 5 fuentes. Corre solo 3×/día.
**Stack:** `Python` · `Pandas` · `DuckDB` · `Parquet`
📖 **[Technical overview](projects/dataforge/README.md)**

### 🛡️ UGPP Shield Pro — Cloud web app para líderes financieros
Aplicación web orientada a CFOs para inteligencia fiscal y de cumplimiento, desplegada en **Cloudflare Pages**.
**Stack:** `JavaScript` · `Vite` · `Cloudflare Pages`

### 🤖 Oportunia — Prospección freelance asistida por IA
Sistema personal que encuentra y califica oportunidades freelance usando IA.
**Stack:** `Python` · `AI / LLM`

### ⚙️ TEMU Automation (RPA) — Bot Playwright
Bot que sube/descarga documentos a un portal logístico emparejando registros por reglas de negocio; elimina trabajo manual repetitivo.
**Stack:** `Python` · `Playwright`

### 🎮 Hearthwood — Juego de Roblox (side project)
Juego cooperativo de supervivencia construido desde cero por fases — arquitectura cliente-servidor en ~23 servicios. Demuestra diseño de sistemas más allá de datos.
**Stack:** `Luau` · `Roblox`

---

## 📈 Currently learning

`AWS (Cloud Practitioner → Solutions Architect)` · `Systems Engineering` · `English → B2` · `dbt`

---

## 📄 CV / Resume

- [CV — Español (PDF)](resume/CV_Juan_Berrio_ES.pdf) · [DOCX](resume/CV_Juan_Berrio_ES.docx)
- [CV — English (PDF)](resume/CV_Juan_Berrio_EN.pdf) · [DOCX](resume/CV_Juan_Berrio_EN.docx)

## 📬 Contact

Open to **remote Cloud / Data Engineering** roles and consulting.
📧 juandyb99@gmail.com · 🔗 [LinkedIn](https://www.linkedin.com/in/juanberrioo)
