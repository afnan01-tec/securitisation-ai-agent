# Securitisation Risk Assessment & Waterfall Engine

### Power BI | Advanced DAX | Financial Risk Analytics | Auto Loan ABS

A Power BI-based financial analytics and risk modeling project designed around an **Auto Loan Asset-Backed Securities (ABS)** portfolio.

The project transforms loan-level portfolio and delinquency data into an interactive risk dashboard using **Power BI, Power Query, Star Schema modeling, and Advanced DAX**.

---

## 🚀 Project Overview

The dashboard covers four major areas:

* Portfolio Performance
* IFRS 9 / Ind AS 109-inspired ECL Analysis
* Cash Flow Waterfall
* Overcollateralisation (OC) Trigger Monitoring

The objective is to demonstrate how Power BI can be used to combine **financial data analytics, credit-risk modeling, and securitisation structure analysis** into a single reporting solution.

---

## 🏗️ Data Model

The Power BI model follows a **Star Schema** architecture.

### Core Tables

| Table                           | Purpose                                  |
| ------------------------------- | ---------------------------------------- |
| `auto_loan_securitisation_data` | Master loan-level portfolio data         |
| `dpd_snapshot_history`          | Monthly loan-level delinquency snapshots |
| `dynamic_loss_monthly`          | Monthly collections and loss data        |
| `static_pool_vintage_data`      | Vintage-level cumulative loss data       |
| `DimDate`                       | Date dimension for time-based analysis   |

---

## 📊 Portfolio Analytics

### Total Portfolio Balance

```DAX
Total Portfolio Balance =
SUM(dpd_snapshot_history[CurrentBalance])
```

### Active Loan Count

```DAX
Active Loan Count =
DISTINCTCOUNT(dpd_snapshot_history[LoanID])
```

### 30+ DPD Balance

```DAX
30+ DPD Balance =
CALCULATE(
    [Total Portfolio Balance],
    dpd_snapshot_history[DPD_Days] >= 30
)
```

### 30+ Delinquency Rate

```DAX
30+ Delinquency Rate =
DIVIDE(
    [30+ DPD Balance],
    [Total Portfolio Balance]
)
```

### Weighted Average Coupon

```DAX
WAC =
DIVIDE(
    SUMX(
        dpd_snapshot_history,
        RELATED(
            auto_loan_securitisation_data[InterestRate]
        ) * dpd_snapshot_history[CurrentBalance]
    ),
    [Total Portfolio Balance]
)
```

### Weighted Average LTV

```DAX
Weighted Avg LTV =
DIVIDE(
    SUMX(
        dpd_snapshot_history,
        RELATED(
            auto_loan_securitisation_data[LTV_Current]
        ) * dpd_snapshot_history[CurrentBalance]
    ),
    [Total Portfolio Balance]
)
```

---

# 🏦 IFRS 9 Expected Credit Loss

The project implements a simplified **IFRS 9 / Ind AS 109-inspired credit-risk staging framework**.

### ECL Formula

```text
ECL = EAD × PD × LGD
```

### Risk Staging

```DAX
IFRS9_Stage =
SWITCH(
    TRUE(),
    dpd_snapshot_history[DPD_Days] >= 90, "Stage 3",
    dpd_snapshot_history[DPD_Days] >= 30, "Stage 2",
    "Stage 1"
)
```

| Stage   |   DPD | Baseline PD |
| ------- | ----: | ----------: |
| Stage 1 |  < 30 |          2% |
| Stage 2 | 30–89 |         15% |
| Stage 3 |  ≥ 90 |        100% |

The model uses a simplified **45% LGD assumption**.

### Total ECL Provision

```DAX
Total_ECL_Provision =
SUMX(
    dpd_snapshot_history,
    dpd_snapshot_history[CurrentBalance]
        * [Baseline_PD]
        * 0.45
)
```

### ECL Coverage Rate

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

### Total Collections

```DAX
Total Collections =
SUM(dynamic_loss_monthly[CollectionsTotal])
```

### Senior Fees

```text
Senior Fees =
Total Portfolio Balance × (0.0050 / 12)
```

### Class A Interest

```text
Class A Interest =
(Total Portfolio Balance × 80%)
× (0.0750 / 12)
```

### Class B Interest

```text
Class B Interest =
(Total Portfolio Balance × 12%)
× (0.1050 / 12)
```

### Residual Spread

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

# 🛡️ Overcollateralisation Trigger

The model monitors the portfolio's modeled **Overcollateralisation (OC) ratio**.

### OC Ratio

```DAX
OC_Ratio =
DIVIDE(
    [Total Portfolio Balance],
    [Total Portfolio Balance] * 0.92
)
```

### Trigger Status

```DAX
OC_Trigger_Status =
IF(
    [OC_Ratio] >= 1.05,
    "PASS - Normal Pay",
    "FAIL - Divert Cash to Class A"
)
```

---

# 📈 Power BI Dashboard

The report contains the following analytical views:

### Executive Summary

* Portfolio Balance
* Active Loan Count
* WAC
* Weighted Average LTV
* 30+ DPD Delinquency Rate

### Risk Staging

* Stage 1 / Stage 2 / Stage 3
* Loan Count
* EAD
* PD
* ECL Provision
* Coverage Rate

### Delinquency Analysis

* 30+ DPD trend
* DPD bucket distribution
* Portfolio delinquency movement

### Vintage Analysis

* Cumulative net loss curves
* Months-on-book analysis
* Vintage performance comparison

### Waterfall & Trigger Monitor

* Collections
* Senior Fees
* Class A Interest
* Class B Interest
* Residual Spread
* OC Ratio
* Trigger Status

---

# 🛠️ Technology

* **Microsoft Power BI**
* **Power Query**
* **DAX**
* **Excel**
* **Star Schema**
* **Financial Risk Analytics**
* **IFRS 9 / Ind AS 109 Concepts**

---

# ⚠️ Disclaimer

This project is developed for **educational, portfolio, and analytical demonstration purposes**.

The ECL assumptions, PD/LGD parameters, waterfall mechanics, and OC calculations are simplified modeling assumptions and do not represent an actual securitisation transaction or regulatory reporting model.

---

## 👤 Author

**Afnan M**

Data Science & AI/ML | Power BI | DAX | Financial Analytics
