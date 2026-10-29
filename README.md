# 🧱 B2B Sales Lakehouse & Analytics Pipeline (Databricks + Tableau)

Praxisnahes Data-Engineering- und Analytics-Projekt mit einer vollständigen **Medallion-Architektur in Databricks (PySpark & Spark SQL)** und einem angebundenen **Tableau Executive Dashboard** für den europäischen B2B-Vertrieb mit Schwerpunkt auf die **DACH-Region**.

---

## 📊 Executive Dashboard Vorschau

Das Dashboard visualisiert Umsatz, Margen, Kundensegmente und Quartalswachstum auf Basis des aggregierten Gold-Layers.

- **Tableau Workbook:** [`tableau/DACH_Executive_Sales_Dashboard.twbx`](tableau/DACH_Executive_Sales_Dashboard.twbx) (inklusive integriertem `.hyper`-Extrakt)

---

## 📁 Datenbasis

Der Datensatz ist **synthetisch generiert** und orientiert sich an realen Vertriebsdaten aus dem B2B-Cloud- & Lizenzgeschäft:
- **Umfang:** 4.591 Transaktionen (Zeitraum: 01.01.2025 – 25.10.2026)
- **Märkte:** 14 Länder in 4 Regionen (**DACH**, Westeuropa, Nordeuropa, Südeuropa)
- **Segmente:** Großkunden (Enterprise), Mittelstand (KMU), Kleingewerbe
- **Produktkategorien:** Cloud Analytics Suite, Datenplattform & ETL, Beratung & Services

---

## 🏛️ Architektur (Medallion Lakehouse)

```
data/executive_sales_gold.csv
        │
        ▼
 [1] Bronze Layer (raw_orders)
     └── Explizites Schema-on-Read, technische Audit-Metadaten (_ingestion_timestamp, _source_file)
        │
        ▼
 [2] Silver Layer (cleaned_orders)
     └── Bereinigung von Whitespaces, Deduplizierung über Bestell_ID,
         Neuberechnung von Jahr, Quartal, Gewinn_EUR & Gewinnmarge,
         automatische Data-Quality-Gates (Assertions)
        │
        ▼
 [3] Gold Layer (executive_sales_gold)
     └── Analytische Data Marts mit gewichteter Margenberechnung je Region, Quartal & Segment
        │
        ▼
 [4] Visual Analytics (Tableau)
     └── DACH Executive Sales Dashboard (Geografischer Drilldown, Segmente & Quartalstrends)
```

In Databricks werden die Tabellen als **Delta-Tabellen im Unity Catalog** gespeichert (`workspace.b2b_vertrieb.*`).

---

## 📐 Kennzahlen & Berechnungsmethodik

| KPI | Formel | Beschreibung |
|---|---|---|
| **Gesamtumsatz (EUR)** | `SUM(Umsatz_EUR)` | Erzielter Bruttoumsatz |
| **Gesamtgewinn (EUR)** | `SUM(Gewinn_EUR)` | Deckungsbeitrag / Reingewinn |
| **Gewinnmarge (%)** | `SUM(Gewinn_EUR) / SUM(Umsatz_EUR)` | **Gewichtete Marge** (betriebswirtschaftlich exakt) |
| **Zielerreichung (%)** | `SUM(Umsatz_EUR) / SUM(Zielumsatz_EUR)` | Erreichungsgrad der Quartalsziele |

> ⚠️ **Methodischer Hinweis zur Margenberechnung:**  
> In diesem Projekt wird die Gesamtmarge bewusst **volumengewichtet** (`SUM(Gewinn) / SUM(Umsatz)`) berechnet. Ein einfaches arithmetisches Mittel (`AVG(Gewinnmarge_Prozent)`) würde kleine und große Aufträge gleich gewichten und die reale Profitabilität verzerren.

---

## 📈 Zentrale Erkenntnisse (Business Insights)

| Region | Aufträge | Umsatz (EUR) | Marge Ø (ungewichtet) | Marge (gewichtet) | Verzerrung |
|---|---:|---:|---:|---:|---:|
| **DACH** | 2.211 | 18.821.161 € | 66,95 % | **65,91 %** | **+1,03 Pp.** |
| **Westeuropa** | 1.126 | 9.312.258 € | 67,19 % | **66,60 %** | +0,59 Pp. |
| **Nordeuropa** | 722 | 6.207.611 € | 66,91 % | **66,08 %** | +0,84 Pp. |
| **Südeuropa** | 532 | 4.575.483 € | 67,37 % | **66,88 %** | +0,49 Pp. |
| **Gesamt** | **4.591** | **38.916.513 €** | **67,05 %** | **66,22 %** | **+0,83 Pp.** |

1. **Umsatztreiber DACH:** Die DACH-Region erwirtschaftet mit 18,8 Mio. € mehr als **48 % des europäischen Gesamtumsatzes**.
2. **Ursache der Margenverzerrung:** Großkunden (Enterprise) verhandeln höhere Rabattstaffeln (durchschnittlich 7,2 % Rabatt), wodurch die reale Marge bei Großaufträgen auf 64,82 % sinkt. Da in der DACH-Region die meisten Großaufträge abgewickelt werden, ist die rechnerische Verzerrung eines einfachen Durchschnitts hier mit **+1,03 Prozentpunkten** am stärksten.

---

## 🗂️ Projektstruktur

```
├── notebooks/
│   └── databricks_medallion_pipeline.ipynb   # Vollständige PySpark-Pipeline (Bronze ➔ Silver ➔ Gold)
├── data/
│   └── executive_sales_gold.csv              # Bereitgestellter Datensatz (4.591 Zeilen)
└── tableau/
    └── DACH_Executive_Sales_Dashboard.twbx   # Tableau Packaged Workbook (.hyper Data Source)
```

---

## 🛠️ Technologien & Tools

- **Data Engineering:** Databricks, PySpark, Spark SQL, Delta Lake, Unity Catalog
- **Business Intelligence:** Tableau Desktop / Tableau Public
- **Sprachen & Formate:** Python 3, SQL, Parquet, Hyper

---

## 🚀 Ausführung & Reproduzierbarkeit

### In Databricks:
1. Notebook [`notebooks/databricks_medallion_pipeline.ipynb`](notebooks/databricks_medallion_pipeline.ipynb) in den Databricks Workspace importieren.
2. Datendatei `data/executive_sales_gold.csv` in ein Unity-Catalog-Volume hochladen (z. B. `/Volumes/workspace/default/rohdaten/`).
3. Cluster starten und Notebook ausführen — Tabellen werden automatisch in Delta Lake persistiert.

### In Tableau:
1. Datei [`tableau/DACH_Executive_Sales_Dashboard.twbx`](tableau/DACH_Executive_Sales_Dashboard.twbx) mit Tableau Desktop oder Tableau Public öffnen.

---

## 📌 Grenzen & Ausblick
- **Datenbeschaffenheit:** Synthetische Daten für Demonstrationszwecke; keine echten internen Finanzdaten.
- **Zeitraum:** 2026 ist bis zum 25.10. abgebildet, weshalb Q4 2026 noch unvollständig ist.
- **Orchestrierung:** Geplante Erweiterung um einen automatisierten Databricks Workflow (Job) mit Triggering und Alerts.
