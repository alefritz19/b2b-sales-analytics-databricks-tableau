# 🚀 End-to-End Lakehouse & BI-Projekt: Databricks + Tableau (DACH & Europa)

Ein praxisnahes Data-Engineering- und Business-Intelligence-Projekt, das eine **Databricks Medallion-Architektur (PySpark & Spark SQL)** mit einem interaktiven **Tableau Public Executive Dashboard** für den europäischen B2B-Markt (Fokus: **DACH-Region** – Deutschland, Österreich, Schweiz) verbindet.

---

## 🏗️ Architektur (Medallion Lakehouse → Tableau BI)

1. **Bronze-Layer (Rohdaten-Ingestion)**: Transaktionale B2B-Cloud- und Lizenzaufträge über europäische Märkte (`DACH`, `Westeuropa`, `Nordeuropa`, `Südeuropa`).
2. **Silver-Layer (PySpark Bereinigung & Anreicherung)**: Deduplizierung, Schema-Validierung, Rabattmodellierung und Berechnung der Nettomargen.
3. **Gold-Layer (`executive_sales_gold.csv` / `.hyper`)**: Aggregiertes analytisches Datenmodell für das **Tableau Executive Dashboard**.

---

## 📊 Wichtige KPIs & Kennzahlen in Tableau

- **Gesamtumsatz (EUR)**: `SUM([Umsatz_EUR])`
- **Gesamtgewinn (EUR)**: `SUM([Gewinn_EUR])`
- **Gewinnmarge (%)**: `AVG([Gewinnmarge_Prozent])`
- **Zielerreichung (%)**: `SUM([Umsatz_EUR]) / SUM([Zielumsatz_EUR])`

---

## 📁 Projektstruktur

- [`DACH_Executive_Sales_Dashboard.twbx`](DACH_Executive_Sales_Dashboard.twbx) — Fertiges Tableau Packaged Workbook inkl. nativem `.hyper`-Extrakt.
- [`notebooks/databricks_medallion_pipeline.ipynb`](notebooks/databricks_medallion_pipeline.ipynb) — PySpark & Spark SQL Transformations-Pipeline (Bronze → Silver → Gold).
- [`data/executive_sales_gold.csv`](data/executive_sales_gold.csv) — Gold-Datensatz (4.579 B2B-Transaktionen, 2025–2026, DACH & Europa).
