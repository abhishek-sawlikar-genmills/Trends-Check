# Trends-Check
Trend Check  notebook performs automated data reconciliation and validation between monthly snapshot layers: - **Current / Target Release**: For e.g July 2026 (`Jul26`) - **Baseline / Prior Historical Release**: June 2026 (`Jun26`)  The checks validate retail sales performance (Value_000 and Volume_ 000 across two major category domains: RTE and RTE_OTHER

## 🔍 Key Notebook Sections & Code Logic

### 1. OGF Check — Conformed Layer
* **Objective:** Compares segment shares (`Global_OGF_Segment`) at each `Global_Market` level between the current conformed fact tables and baseline tables.
* **Tables Queried:**
  * Current: `glbl_cpw_prod.conformed.factretailsales`, `dimproduct`, `dimmarket`, `dimperiod`, `metadata.forex`, `glbl_cpw_prod.adhoc.period_mat_ytd`
  * Prior: `glbl_cpw_prod.history_conformed.factretailsales_Jun26`, `dimproduct_Jun26`, `dimmarket_Jun26`, `dimperiod_Jun26`, `period_mat_ytd_Jun26`
* **Filtering Conditions:**
  * `Global_Total_Mkt_Flag = 'Y'`
  * Categories: `'RTE'`, `'RTE_OTHER'`
  * Calendar years: `2022` through `2026`
  * MAT filter: `pmt.MAT_Flag IN ('MAT TY', 'MAT LY', 'MAT 2LY')`
  * Excluded local markets: Non-standard/scan markets (e.g., `'Baltics'`, `'France_Scan'`, `'Brazil_Scan'`, `'Greece_C&C'`, etc.).
* **Calculations:**
  * Uses SQL window functions `OVER (PARTITION BY gm.Global_Market)` to calculate market totals in a single pass.
  * Measures segment percentage share differences: $\text{Jul26 \%} - \text{Jun26 \%}$.
  * Appends an overall grand total row via `UNION ALL`.
 
### 2. OGF Check — Derived Layer
* **Objective:** Validates whether the reporting metrics maintain parity after transformation into aggregated reporting index tables.
* **Tables Queried:**
  * Current: `glbl_cpw_prod.derived.retailindextotal`
  * Prior: `glbl_cpw_prod.history_conformed.retailindextotal_Jun26`
* **Logic:** Computes market segment aggregates (`Jul26_SegmentAggregates`), compares with per-database totals, and checks for discrepancy between conformed and derived outputs.

### 3. Manufacturer Check — Conformed & Derived Layers
* **Objective:** Tracks manufacturer market share shifts (e.g., `KELLOGGS`, `NESTLE`, `PRIVATE LABEL`, `SANITARIUM`, `WEETABIX`) within each market between releases.
* **Key Metrics:**
  * `Sum_of_Value_000_CHF_Jul26` vs `Sum_of_Value_000_CHF_Jun26`
  * `Sum_of_Volume_000_Jul26` vs `Sum_of_Volume_000_Jun26`
  * Percentage delta on Value and Volume.
