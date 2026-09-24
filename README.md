# Securitisation Risk Assessment & Waterfall Engine

### Power BI | Advanced DAX | Financial Risk Analytics | Auto Loan ABS

An end-to-end **Power BI financial analytics and risk modeling project** built around an **Auto Loan Asset-Backed Securities (ABS)** portfolio.

The project transforms loan-level portfolio, delinquency, collection, loss, and vintage data into an interactive risk dashboard using **Power Query, Star Schema modeling, DAX, IFRS 9-inspired ECL analysis, cash-flow waterfall modeling, and Overcollateralisation (OC) trigger monitoring**.

---

## 📌 Project Overview

The project analyzes an Auto Loan ABS portfolio from multiple perspectives:

* Portfolio performance
* Loan delinquency
* Credit-risk staging
* Expected Credit Loss (ECL)
* Vintage loss performance
* Monthly cash collections
* Cash-flow waterfall allocation
* Overcollateralisation trigger monitoring

The goal is to demonstrate how **Power BI and DAX can be used to build a financial risk analytics solution from raw loan-level data to an executive-level dashboard**.

---

## 🏗️ Data Model

The Power BI model follows a **Star Schema** architecture.

### Core Data Sources

| Dataset                              | Purpose                                              |
| ------------------------------------ | ---------------------------------------------------- |
| `auto_loan_securitisation_data.xlsx` | Master loan-level portfolio data                     |
| `dpd_snapshot_history.xlsx`          | Monthly loan-level delinquency and balance snapshots |
| `dynamic_loss_monthly.xlsx`          | Monthly collections and loss performance             |
| `static_pool_vintage_data.xlsx`      | Vintage-level cumulative loss performance            |

### Power BI Model

```text
                         DimDate
                            |
            +---------------+---------------+
            |               |               |
            v               v               v
   DPD Snapshot       Dynamic Loss      Static Pool
      History           Monthly          Vintage Data
            |
            |
            v
   Auto Loan Master Data
```

---

# 📊 Portfolio Analytics

The project uses DAX measures to calculate key portfolio performance indicators.

## Total Portfolio Balance

```DAX
Total Portfolio Balance =
SUM(dpd_snapshot_history[CurrentBalance])
```

## Active Loan Count

```DAX
Active Loan Count =
DISTINCTCOUNT(dpd_snapshot_history[LoanID])
```

## 30+ DPD Balance

```DAX
30+ DPD Balance =
CALCULATE(
    [Total Portfolio Balance],
    dpd_snapshot_history[DPD_Days] >= 30
)
```

## 30+ Delinquency Rate

```DAX
30+ Delinquency Rate =
DIVIDE(
    [30+ DPD Balance],
    [Total Portfolio Balance]
)
```

---

# 📈 Weighted Portfolio Metrics

## Weighted Average Coupon (WAC)

```DAX
WAC =
DIVIDE(
    SUMX(
        dpd_snapshot_history,
        RELATED(
            auto_loan_securitisation_data[InterestRate]
        ) *
        dpd_snapshot_history[CurrentBalance]
    ),
    [Total Portfolio Balance]
)
```

## Weighted Average LTV

```DAX
Weighted Avg LTV =
DIVIDE(
    SUMX(
        dpd_snapshot_history,
        RELATED(
            auto_loan_securitisation_data[LTV_Current]
        ) *
        dpd_snapshot_history[CurrentBalance]
    ),
    [Total Portfolio Balance]
)
```

---

# 🏦 IFRS 9 Expected Credit Loss (ECL)

The project implements a simplified **IFRS 9 / Ind AS 109-inspired credit-risk staging framework**.

The basic ECL relationship used is:

```text
ECL = EAD × PD × LGD
```

For portfolio-level analysis:

```text
Total ECL = Σ(EAD × PD × LGD)
```

## IFRS 9 Staging

Loans are classified based on their delinquency status.

```DAX
IFRS9_Stage =
SWITCH(
    TRUE(),
    dpd_snapshot_history[DPD_Days] >= 90, "Stage 3",
    dpd_snapshot_history[DPD_Days] >= 30, "Stage 2",
    "Stage 1"
)
```

### Stage Assumptions

| Stage   | DPD Condition | Baseline PD |
| ------- | ------------: | ----------: |
| Stage 1 |      < 30 DPD |          2% |
| Stage 2 |     30–89 DPD |         15% |
| Stage 3 |      ≥ 90 DPD |        100% |

The model uses a simplified **45% LGD assumption**.

## Baseline PD

```DAX
Baseline_PD =
SWITCH(
    SELECTEDVALUE(dpd_snapshot_history[IFRS9_Stage]),
    "Stage 1", 0.02,
    "Stage 2", 0.15,
    "Stage 3", 1.00,
    0.02
)
```

## Total ECL Provision

```DAX
Total_ECL_Provision =
SUMX(
    dpd_snapshot_history,
    dpd_snapshot_history[CurrentBalance]
        * [Baseline_PD]
        * 0.45
)
```

## ECL Coverage Rate

```DAX
ECL_Coverage_Rate =
DIVIDE(
    [Total_ECL_Provision],
    [Total_EAD]
)
```

---

# 💰 Cash Flow Waterfall

The project models a simplified securitisation cash-flow waterfall.

```text
Gross Collections
        ↓
Senior Fees
        ↓
Net Collections
        ↓
Class A Interest
        ↓
Class B Interest
        ↓
Residual Equity Spread
```

## Total Collections

```DAX
Total Collections =
SUM(dynamic_loss_monthly[CollectionsTotal])
```

## Senior Fees

The model assumes a 50 bps annual servicing/trustee fee.

```text
Senior Fees =
Total Portfolio Balance × (0.0050 / 12)
```

## Net Collections

```text
Net Collections =
MAX(0, Total Collections - Senior Fees)
```

## Class A Interest

```text
Class A Interest =
(Total Portfolio Balance × 80%)
× (0.0750 / 12)
```

## Class B Interest

```text
Class B Interest =
(Total Portfolio Balance × 12%)
× (0.1050 / 12)
```

## Residual Equity Spread

```DAX
Residual_Spread =
MAX(
    0,
    [Net_Collections]
        - [ClassA_Interest]
        - [ClassB_Interest]
)
```

---

# 🛡️ Overcollateralisation (OC) Trigger

The model monitors a simplified **Overcollateralisation ratio** to evaluate structural protection.

## OC Ratio

```DAX
OC_Ratio =
DIVIDE(
    [Total Portfolio Balance],
    [Total Portfolio Balance] * 0.92
)
```

## OC Trigger Status

```DAX
OC_Trigger_Status =
IF(
    [OC_Ratio] >= 1.05,
    "PASS - Normal Pay",
    "FAIL - Divert Cash to Class A"
)
```

### Trigger Logic

```text
OC Ratio ≥ 105%
       ↓
PASS
Normal Pay

OC Ratio < 105%
       ↓
FAIL
Divert Cash to Class A
```

---

# 📊 Power BI Dashboard

The report contains multiple analytical sections.

### Executive Summary

Key portfolio KPIs:

* Total Portfolio Balance
* Active Loan Count
* Weighted Average Coupon
* Weighted Average LTV
* 30+ DPD Delinquency Rate

### Risk Staging Matrix

Provides visibility into:

* Stage 1
* Stage 2
* Stage 3
* Loan Count
* EAD
* PD
* ECL Provision
* ECL Coverage Rate

### Delinquency Analysis

Visualizes:

* 30+ DPD delinquency trend
* DPD bucket distribution
* Current vs delinquent balances

### Vintage Loss Performance

Analyzes:

* Cumulative net loss
* Months on Book
* Vintage-level performance

### Waterfall & Trigger Monitor

Displays:

* Total Collections
* Senior Fees
* Net Collections
* Class A Interest
* Class B Interest
* Residual Spread
* OC Ratio
* OC Trigger Status

---

# 🧮 DAX Analytics

The project uses DAX for:

* Portfolio KPIs
* Delinquency calculations
* Weighted averages
* IFRS 9 staging
* PD calculation
* ECL provisioning
* Cash-flow waterfall
* OC trigger monitoring

The complete DAX implementation is available in:

`dax/measures.dax`

---

# 🛠️ Technology Stack

* **Microsoft Power BI**
* **Power Query**
* **DAX**
* **Microsoft Excel**
* **Star Schema**
* **Financial Risk Analytics**
* **IFRS 9 / Ind AS 109 Concepts**

---

# 📂 Repository Structure

```text
Securitisation-Risk-Assessment/
│
├── README.md
│
├── Securitisation_Risk_Assessment.pbix
│
├── dax/
│   └── measures.dax
│
└── data/
    ├── auto_loan_securitisation_data.xlsx
    ├── dpd_snapshot_history.xlsx
    ├── dynamic_loss_monthly.xlsx
    └── static_pool_vintage_data.xlsx
```

---

# 🔄 Project Workflow

```text
Excel Data Sources
       ↓
Power Query
       ↓
Data Cleaning & Transformation
       ↓
Star Schema
       ↓
DAX Measures & Calculated Columns
       ↓
Portfolio Analytics
       ↓
ECL Risk Analysis
       ↓
Cash Flow Waterfall
       ↓
OC Trigger Monitoring
       ↓
Power BI Dashboard
```

---

# ⚠️ Assumptions & Disclaimer

This project is developed for **educational, portfolio, and analytical demonstration purposes**.

The ECL parameters, PD/LGD assumptions, waterfall mechanics, and OC trigger calculations are simplified modeling assumptions. They are not intended to represent an actual securitisation transaction, investment recommendation, accounting conclusion, or regulatory reporting model.

---

## 👤 Author

**Afnan M**

Data Science & AI/ML | Power BI | DAX | Financial Analytics
