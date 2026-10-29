# 🧱 B2B Sales Lakehouse & Analytics Pipeline (Databricks + Tableau)

Praxisnahes Data-Engineering- und Analytics-Projekt mit einer vollständigen **Medallion-Architektur in Databricks (PySpark & Spark SQL)** und einem angebundenen **Tableau Executive Dashboard** für den europäischen B2B-Vertrieb mit Schwerpunkt auf die **DACH-Region**.

---

## 📊 Executive Dashboard Vorschau

Das Dashboard visualisiert Umsatz, Margen, Kundensegmente und Quartalswachstum auf Basis des aggregierten Gold-Layers.

- **Tableau Workbook:** [`tableau/DACH_Executive_Sales_Dashboard.twbx`](tableau/DACH_Executive_Sales_Dashboard.twbx) (inklusive integriertem `.hyper`-Extrakt)

---

## 📁 Datenbasis

Der Datensatz ist **synthetisch generiert** und orientiert sich an realen Vertriebsdaten aus dem B2B-Cloud- & Lizenzgeschäft:
- **Quelldatei:** [`data/b2b_sales_raw.csv`](data/b2b_sales_raw.csv)
- **Umfang:** 4.591 Transaktionen (Zeitraum: 01.01.2025 – 25.10.2026)
- **Märkte:** 14 Länder in 4 Regionen (**DACH**, Westeuropa, Nordeuropa, Südeuropa)
- **Segmente:** Großkunden (Enterprise), Mittelstand (KMU), Kleingewerbe
- **Produktkategorien:** Cloud Analytics Suite, Datenplattform & ETL, Beratung & Services

---

## 🏛️ Architektur (Medallion Lakehouse)

```
data/b2b_sales_raw.csv (Quelldaten)
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
     └── Analytische Data Marts mit gewichteter Margenberechnung und Drilldowns (Region × Kundensegment)
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
| **Deckungsbeitrag (EUR)** | `SUM(Gewinn_EUR)` | Erwirtschafteter Deckungsbeitrag (Umsatz minus direkte Kosten) |
| **Gewinnmarge (%)** | `SUM(Gewinn_EUR) / SUM(Umsatz_EUR)` | **Volumengewichtete Marge** (betriebswirtschaftlich exakt) |
| **Zielerreichung (%)** | `SUM(Umsatz_EUR) / SUM(Zielumsatz_EUR)` | Erreichungsgrad der Quartalsziele |

> ⚠️ **Methodischer Hinweis zur Margenberechnung:**  
> In diesem Projekt wird die Gesamtmarge bewusst **volumengewichtet** (`SUM(Gewinn) / SUM(Umsatz)`) berechnet. Ein einfaches arithmetisches Mittel (`AVG(Gewinnmarge_Prozent)`) würde kleine und große Aufträge gleich gewichten und die reale Profitabilität verzerren.

---

## 📈 Zentrale Erkenntnisse & Nachweis der Margenverzerrung

### 1. Regionaler Margenvergleich:
| Region | Aufträge | Umsatz (EUR) | Marge Ø (ungewichtet) | Marge (gewichtet) | Verzerrung |
|---|---:|---:|---:|---:|---:|
| **DACH** | 2.211 | 18.821.161 € | 66,95 % | **65,91 %** | **+1,03 Pp.** |
| **Westeuropa** | 1.126 | 9.312.258 € | 67,19 % | **66,60 %** | +0,59 Pp. |
| **Nordeuropa** | 722 | 6.207.611 € | 66,91 % | **66,08 %** | +0,84 Pp. |
| **Südeuropa** | 532 | 4.575.483 € | 67,37 % | **66,88 %** | +0,49 Pp. |
| **Gesamt** | **4.591** | **38.916.513 €** | **67,05 %** | **66,22 %** | **+0,83 Pp.** |

### 2. Nachweis über Region × Kundensegment:
Warum ist der Verzerrungseffekt in der **DACH-Region** mit **+1,03 Prozentpunkten** am stärksten?

| Region | Kundensegment | Aufträge | Segment-Umsatz (EUR) | **Umsatzanteil Region** | **Marge gewichtet** |
|---|---|---:|---:|---:|---:|
| **DACH** | **Großkunden (Enterprise)** | **1.006** | **12.240.290 €** | **65,03 %** | **65,32 %** |
| DACH | Mittelstand (KMU) | 833 | 5.373.522 € | 28,55 % | 66,57 % |
| DACH | Kleingewerbe | 372 | 1.207.349 € | 6,41 % | 69,03 % |
| **Nordeuropa** | Großkunden (Enterprise) | 296 | 3.677.183 € | 59,24 % | 65,37 % |
| **Westeuropa** | Großkunden (Enterprise) | 486 | 5.679.326 € | 60,99 % | 66,08 % |
| **Südeuropa** | Großkunden (Enterprise) | 242 | 2.957.831 € | 64,65 % | 67,02 % |

- **Empirischer Befund:** Die DACH-Region weist mit **65,03 % den höchsten Enterprise-Umsatzanteil** aller 4 Regionen auf.
- **Wirkungsmechanismus:** Großkunden verhandeln höhere Rabattstaffeln, weshalb die Marge hier mit 65,32 % am niedrigsten liegt. Da Großkunden 65 % des Umsatzes, aber nur 45,5 % der Aufträge ausmachen, überbewertet das einfache arithmetische Mittel `AVG(Marge)` die margenstarken Kleinaufträge (69,03 %) unverhältnismäßig stark. In DACH führt dieses Ungleichgewicht zur größten Abweichung (+1,03 Pp.).

---

## 🗂️ Projektstruktur

```
├── notebooks/
│   └── databricks_medallion_pipeline.ipynb   # Vollständige PySpark-Pipeline (Bronze ➔ Silver ➔ Gold)
├── data/
│   └── b2b_sales_raw.csv                     # Quelldatensatz (4.591 Zeilen)
└── tableau/
    └── DACH_Executive_Sales_Dashboard.twbx   # Tableau Packaged Workbook (.hyper Data Source)
```

---

## 🛠️ Technologien & Tools

- **Data Engineering:** Databricks, PySpark, Spark SQL, Delta Lake, Unity Catalog
- **Business Intelligence:** Tableau Desktop / Tableau Public
- **Formate & Schnittstellen:** Python 3, SQL, CSV, Hyper

---

## 🚀 Ausführung & Reproduzierbarkeit

### In Databricks:
1. Notebook [`notebooks/databricks_medallion_pipeline.ipynb`](notebooks/databricks_medallion_pipeline.ipynb) in den Databricks Workspace importieren.
2. Datendatei [`data/b2b_sales_raw.csv`](data/b2b_sales_raw.csv) in ein Unity-Catalog-Volume hochladen (z. B. `/Volumes/workspace/default/rohdaten/`).
3. Cluster starten und Notebook ausführen — Tabellen werden automatisch in Delta Lake persistiert.

### In Tableau:
1. Datei [`tableau/DACH_Executive_Sales_Dashboard.twbx`](tableau/DACH_Executive_Sales_Dashboard.twbx) mit Tableau Desktop oder Tableau Public öffnen.

---

## 📌 Grenzen & Methodik
- **Datenbeschaffenheit:** Synthetischer Datensatz zur Demonstration professioneller Best Practices im Cloud Data Engineering.
- **Zeitraum:** 2026 ist bis zum 25.10. abgebildet, weshalb Q4 2026 noch unvollständig ist.
- **Orchestrierung:** Geplante Erweiterung um einen automatisierten Databricks Workflow (Job) mit Monitoring.
