# Economic Growth Zone (EGZ) Identification Project

This project implements a data-driven methodology to identify **Economic Growth Zones (EGZ)** across India. It analyzes Aadhaar enrollment and demographic update data to detect "in-migration hotspots"—rural or semi-urban areas where the influx of adults (indicated by address updates) significantly outpaces new enrollments.

---

## 🔬 Methodology & Calculation Logic

The analysis follows a specific logical pipeline to ensure statistical validity and meaningful insights.

### 1. Data Ingestion & Pre-processing
*   **Input**: Raw Aadhaar CSV files for Enrollments and Demographic Updates.
*   **Logic**: Aggregates data by `Pincode` + `Year`.
    *   *Enrollments*: Sum of new registrations (`age_0_5` + `age_5_17` + `age_18_greater`).
    *   *Updates*: Sum of demographic changes (`updates_5_17` + `updates_18_plus`).

### 2. The Core Metrics
We calculate two primary ratios for every pincode:

*   **UER (Update-to-Enrollment Ratio)**:
    ```
    UER = Total Updates / Total Enrollments
    ```
    *Why*: Measures overall activity relative to organic growth. A high UER means more people are "arriving" or "updating" than "being born/registered".

*   **UER 18+ (Adult Dominant Ratio)**:
    ```
    UER_18+ = Updates (18+) / Total Enrollments
    ```
    *Why*: High updates by adults specifically indicate **economic migration** (workers moving for jobs) rather than general population shuffling.

*   **NMR (Net Migration Ratio)**:
    ```
    NMR = UER - 1
    ```
    *Why*: Standardizes the metric around zero.
    *   `NMR > 0`: Net Attraction (In-migration).
    *   `NMR < 0`: Net Outflow.
    *   `NMR > 5`: Extreme Attraction (5x more updates than enrollments).

### 3. The EGZ Filter (Candidate Selection)
A pincode is flagged as a "Growth Zone" **only if**:
1.  **Validity**: New Enrollments ≥ 30 (to avoid statistical noise from tiny villages).
2.  **Dual Threshold**:
    *   `UER` > 90th Percentile of all pincodes (Top 10% activity).
    *   `UER_18+` > 90th Percentile of all pincodes (Top 10% adult influx).

---

## 📂 Project Structure & File Manifest

### 1. Python Analysis Scripts (Run in this order)
| Script | Created Why? | Functionality |
| :--- | :--- | :--- |
| **1. `analyze_egz.py`** | **Foundation**. Created to establish the basic logic and verify if the methodology yields any results at all. | • Loads raw CSVs.<br>• Calculates UER & UER 18+.<br>• Applies 90th percentile filters.<br>• Outputs the first raw candidate list. |
| **2. `analyze_egz_enhanced.py`** | **Context**. The raw list was just numbers. We needed to know *where* these places are (State/District). | • Merges state/district metadata.<br>• Aggregates counts by region.<br>• Creates the first human-readable report. |
| **3. `analyze_egz_detailed.py`** | **Depth**. We needed to quantify *how strong* the attraction is. Is it just "high" or "extreme"? | • Introduces **Net Migration Ratio (NMR)**.<br>• Categorizes areas (Stable, High Attraction, Extreme).<br>• Flags 60+ extreme outliers (NMR > 50). |
| **4. `analyze_egz_statistical.py`** | **Validation**. To prove the findings aren't flukes, we needed statistical rigor. | • Calculates distribution quartiles (Q1, Median, Q3).<br>• Runs correlations (proven: Adult migration drives the trend).<br>• Detects 3-sigma anomalies. |
| **5. `analyze_egz_rural.py`** | **Focus**. The user hypothesis was that growth is rural. We needed to prove/disprove this. | • Downloads India Post data.<br>• Classifies pincodes as Rural (B.O.) vs Urban (H.O.).<br>• Filters list to show only Rural candidates (86% matched!). |

### 2. Output Data Files (CSV)
| File | Contains | Why Use It? |
| :--- | :--- | :--- |
| `egz_full_statistical_analysis.csv` | **Everything**. All pincodes, all metrics, all labels. | Use for Machine Learning or deep custom analysis. The master dataset. |
| `egz_candidates_with_location.csv` | **The Winners**. Only the 1,556 pincodes that passed the EGZ filter. | Use to identify specific investment targets or hotpots. |
| `egz_candidates_rural.csv` | **The Villages**. Subset of 1,335 rural "winners". | Use if your focus is strictly on Rural Development or Rural Banking. |
| `egz_district_nmr_analysis.csv` | District rankings. | Use to find which *districts* are booming (e.g., Thoubal, Manipur). |

### 3. Human-Readable Reports (Markdown)
| File | Purpose |
| :--- | :--- |
| **`egz_final_report.md`** | **The Executive Summary**. Read this first. Contains all key stats, logic, and conclusions. |
| `egz_rural_report.md` | Specific insights on the rural segment of the analysis. |
| `egz_detailed_analysis.md` | Breakdown of the Net Migration Ratio categories. |

---

## 🚀 Execution Guide

To reproduce the entire analysis from scratch:

```bash
# Step 1: Core Identification
python3 analyze_egz.py

# Step 2: Add Location Context
python3 analyze_egz_enhanced.py

# Step 3: Calculate NMR & Categories
python3 analyze_egz_detailed.py

# Step 4: Run Statistical Validation
python3 analyze_egz_statistical.py

# Step 5: Filter for Rural Areas
python3 analyze_egz_rural.py
```

## 📊 Key Findings Summary
*   **1,556** pincodes identified as high-growth zones.
*   **85.8% (1,335)** of these are **Rural** locations.
*   **Chandigarh** & **Manipur** show the highest state-level attraction signals.
*   **Correlation**: Adult migration correlates **99.5%** with overall migration, proving that economic movement is the primary driver.
