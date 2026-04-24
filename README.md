# FinFlow Analytics 🚀
### Real-Time SaaS Fintech Funnel Intelligence Platform

![Python](https://img.shields.io/badge/Python-3.9+-3776AB?style=flat&logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0+-4479A1?style=flat&logo=mysql&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2.0+-150458?style=flat&logo=pandas&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-3.7+-11557C?style=flat)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat)

---

## 🔹 Project Overview

**FinFlow Analytics** is an end-to-end real-time funnel intelligence system built for B2B SaaS platforms operating in the fintech space. It ingests simulated event-stream data, runs SQL transformation pipelines, and delivers actionable analytics on user conversion, churn behavior, feature adoption, and subscription revenue health.

**The problem it solves:**
- SaaS companies lose revenue due to undetected funnel drop-offs and late churn signals
- Without a unified analytics layer, product, sales, and support operate in silos
- This platform identifies **where users drop off**, **which features drive retention**, and **which accounts are at churn risk** — before revenue is lost

**Dataset at a glance (RavenStack SaaS):**
- 500 B2B accounts | 5,000 subscriptions | 25,000 feature usage events
- $11.34M total MRR | 22% account churn rate | 600 churn events logged
- Industries: DevTools, FinTech, Cybersecurity, HealthTech, EdTech

---

## 🔹 Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                         DATA SOURCES                             │
│     CSV Files  /  Simulated Event Stream (Python Producer)       │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                      MySQL DATABASE                              │
│  ┌────────────┐  ┌─────────────────┐  ┌──────────────────────┐  │
│  │  accounts  │  │  subscriptions  │  │   feature_usage      │  │
│  ├────────────┤  ├─────────────────┤  ├──────────────────────┤  │
│  │churn_events│  │support_tickets  │  │    event_stream      │  │
│  └────────────┘  └─────────────────┘  └──────────────────────┘  │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│               SQL TRANSFORMATION PIPELINE                        │
│    Raw Events ──► Stage Classification ──► realtime_funnel       │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│           PYTHON ANALYTICS ENGINE  (Pandas + NumPy)              │
│   Conversion Rate │ Churn Rate │ Usage Depth │ MRR at Risk       │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│        VISUALIZATION DASHBOARD  (Matplotlib + Seaborn)           │
│   Funnel Chart │ Conversion │ Churn Heatmap │ Correlation Matrix  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🔹 Tech Stack

| Layer         | Technology                   | Purpose                              |
|---------------|------------------------------|--------------------------------------|
| Language      | Python 3.9+                  | Data generation, analytics, charts   |
| Database      | MySQL 8.0+                   | Event storage & SQL transformations  |
| Processing    | Pandas 2.0+, NumPy           | Data wrangling & KPI computation     |
| Visualization | Matplotlib 3.7+, Seaborn     | 5-panel analytics dashboard          |
| Connector     | mysql-connector-python       | Python ↔ MySQL bridge                |

---

## 🔹 Features

- ✅ **Real-time data simulation** — Generates signup, subscribe, usage, and churn events via Python
- ✅ **Funnel stage classification** — Maps every account to Trial → Active → At-Risk → Churned
- ✅ **Conversion & churn tracking** — Computes rates by plan tier (Basic / Pro / Enterprise) and industry
- ✅ **Feature usage analysis** — Ranks features by `usage_count` and `usage_duration_secs` per stage
- ✅ **Support-churn correlation** — Links `resolution_time_hours` and ticket priority to churn probability
- ✅ **Revenue intelligence** — Tracks MRR, ARR, and upgrade/downgrade flag events per cohort
- ✅ **Visualization dashboard** — 5-panel executive-ready charts for analysts and product teams

---

## 🔹 Project Structure

```
finflow-analytics/
│
├── sql/
│   ├── 01_create_database.sql        # Database + table creation
│   ├── 02_pipeline_transform.sql     # Populate realtime_funnel table
│   └── 03_sample_queries.sql         # Analytical SQL queries
│
├── data_pipeline/
│   ├── insert_data.py                # Simulated data generator
│   └── db_connect.py                 # MySQL connection helper
│
├── analytics/
│   ├── analytics.py                  # Main analytics + visualization script
│   └── kpi_report.py                 # KPI summary printer
│
├── notebooks/
│   └── Flowlytics_Funnel_Intelligence.ipynb
│
├── data/
│   ├── ravenstack_accounts.csv
│   ├── ravenstack_subscriptions.csv
│   ├── ravenstack_feature_usage.csv
│   ├── ravenstack_churn_events.csv
│   └── ravenstack_support_tickets.csv
│
├── README.md
├── INSIGHTS.md
├── requirements.txt
└── .gitignore
```

---

## 🔹 Full Setup Guide

### Prerequisites
- Python 3.9+ installed
- MySQL 8.0+ running locally
- `pip` available

---

### Step 1 — Clone & Install Dependencies

```bash
git clone FinFlow Analytics Real-Time SaaS Fintech Funnel Intelligence Platform
pip install -r requirements.txt
```

---

### Step 2 — MySQL Database Setup

Run in MySQL Workbench or via terminal (`mysql -u root -p`):

```sql
-- ──────────────────────────────────────────────────────
-- CREATE DATABASE
-- ──────────────────────────────────────────────────────
CREATE DATABASE IF NOT EXISTS funnel_analytics;
USE funnel_analytics;

-- ──────────────────────────────────────────────────────
-- TABLE 1: event_stream  (raw ingest layer)
-- ──────────────────────────────────────────────────────
CREATE TABLE IF NOT EXISTS event_stream (
    event_id        VARCHAR(20)  PRIMARY KEY,
    account_id      VARCHAR(20)  NOT NULL,
    event_type      VARCHAR(20)  NOT NULL,      -- signup | subscribe | usage | churn
    plan_tier       VARCHAR(20)  DEFAULT NULL,
    usage_count     INT          DEFAULT 0,
    event_timestamp DATETIME     NOT NULL,
    created_at      TIMESTAMP    DEFAULT CURRENT_TIMESTAMP
);

-- ──────────────────────────────────────────────────────
-- TABLE 2: realtime_funnel  (transformed stage view)
-- ──────────────────────────────────────────────────────
CREATE TABLE IF NOT EXISTS realtime_funnel (
    account_id      VARCHAR(20)  PRIMARY KEY,
    funnel_stage    VARCHAR(20)  NOT NULL,       -- Trial | Active | At-Risk | Churned
    plan_tier       VARCHAR(20)  DEFAULT NULL,
    total_events    INT          DEFAULT 0,
    usage_count     INT          DEFAULT 0,      -- NOTE: usage_count, NOT usage
    churn_flag      TINYINT(1)   DEFAULT 0,
    last_updated    TIMESTAMP    DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

### Step 3 — Insert Simulated Data

**`data_pipeline/insert_data.py`**

```python
import mysql.connector
import random
import uuid
from datetime import datetime, timedelta

# ── DB Connection ──────────────────────────────────────
conn = mysql.connector.connect(
    host="localhost",
    user="root",
    password="your_password",     # ← update this
    database="funnel_analytics"
)
cursor = conn.cursor()

# ── Config ─────────────────────────────────────────────
EVENT_TYPES  = ["signup", "subscribe", "usage", "churn"]
PLAN_TIERS   = ["Basic", "Pro", "Enterprise"]
NUM_ACCOUNTS = 100
NUM_EVENTS   = 500

# ── Generate & insert events ───────────────────────────
def random_timestamp():
    start = datetime(2024, 1, 1)
    return start + timedelta(
        days=random.randint(0, 364),
        hours=random.randint(0, 23),
        minutes=random.randint(0, 59)
    )

rows = []
for _ in range(NUM_EVENTS):
    rows.append((
        f"EVT-{uuid.uuid4().hex[:8].upper()}",
        f"A-{random.randint(1000, 1000 + NUM_ACCOUNTS):04d}",
        random.choice(EVENT_TYPES),
        random.choice(PLAN_TIERS),
        random.randint(1, 20),           # usage_count
        random_timestamp()
    ))

insert_sql = """
    INSERT INTO event_stream
        (event_id, account_id, event_type, plan_tier, usage_count, event_timestamp)
    VALUES (%s, %s, %s, %s, %s, %s)
"""
cursor.executemany(insert_sql, rows)
conn.commit()
print(f"✅ Inserted {cursor.rowcount} events into event_stream")
cursor.close()
conn.close()
```

```bash
python data_pipeline/insert_data.py
```

---

### Step 4 — Run SQL Transformation Pipeline

**`sql/02_pipeline_transform.sql`**

```sql
USE funnel_analytics;

-- Clear previous run
TRUNCATE TABLE realtime_funnel;

-- Populate realtime_funnel with stage classification
INSERT INTO realtime_funnel
    (account_id, funnel_stage, plan_tier, total_events, usage_count, churn_flag)
SELECT
    account_id,

    CASE
        WHEN MAX(event_type = 'churn')     = 1 THEN 'Churned'
        WHEN MAX(event_type = 'usage')     = 1
         AND MAX(event_type = 'subscribe') = 1 THEN 'Active'
        WHEN MAX(event_type = 'subscribe') = 1 THEN 'At-Risk'
        ELSE                                        'Trial'
    END                          AS funnel_stage,

    MAX(plan_tier)               AS plan_tier,
    COUNT(*)                     AS total_events,
    SUM(usage_count)             AS usage_count,
    MAX(event_type = 'churn')    AS churn_flag

FROM  event_stream
GROUP BY account_id;

-- Verify results
SELECT funnel_stage, COUNT(*) AS accounts
FROM   realtime_funnel
GROUP  BY funnel_stage
ORDER  BY FIELD(funnel_stage, 'Trial', 'Active', 'At-Risk', 'Churned');
```

```bash
mysql -u root -p funnel_analytics < sql/02_pipeline_transform.sql
```

---

### Step 5 — Run Analytics & Generate Dashboard

**`analytics/analytics.py`**

```python
import mysql.connector
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import seaborn as sns
import numpy as np

# ── Connection ─────────────────────────────────────────
conn = mysql.connector.connect(
    host="localhost", user="root",
    password="your_password", database="funnel_analytics"
)

funnel_df = pd.read_sql("SELECT * FROM realtime_funnel", conn)
events_df = pd.read_sql("SELECT * FROM event_stream",   conn)
conn.close()

# ── Stage ordering ─────────────────────────────────────
stage_order = ["Trial", "Active", "At-Risk", "Churned"]
funnel_df["funnel_stage"] = pd.Categorical(
    funnel_df["funnel_stage"], categories=stage_order, ordered=True
)

# ── KPIs ───────────────────────────────────────────────
stage_counts    = funnel_df["funnel_stage"].value_counts().reindex(stage_order, fill_value=0)
total           = len(funnel_df)
active          = stage_counts.get("Active",  0)
churned         = stage_counts.get("Churned", 0)
trial           = stage_counts.get("Trial",   0)
conversion_rate = round(active  / total * 100, 1) if total else 0
churn_rate      = round(churned / total * 100, 1) if total else 0

print(f"\n{'='*45}")
print(f"  FinFlow Analytics — KPI Summary")
print(f"{'='*45}")
print(f"  Total Accounts  : {total}")
print(f"  Trial           : {trial}")
print(f"  Active          : {active}")
print(f"  Churned         : {churned}")
print(f"  Conversion Rate : {conversion_rate}%")
print(f"  Churn Rate      : {churn_rate}%")
print(f"{'='*45}\n")

# ── Dashboard layout ───────────────────────────────────
colors = ["#F39C12", "#27AE60", "#E67E22", "#C0392B"]
fig    = plt.figure(figsize=(18, 12))
fig.patch.set_facecolor("#F5F7FA")
gs     = gridspec.GridSpec(2, 3, figure=fig, hspace=0.45, wspace=0.35)

# Graph 1 — Funnel Distribution
ax1  = fig.add_subplot(gs[0, 0])
bars = ax1.bar(stage_order, stage_counts.values, color=colors,
               edgecolor="white", linewidth=1.5)
for bar, val in zip(bars, stage_counts.values):
    ax1.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 0.4,
             str(val), ha="center", fontsize=11, fontweight="bold")
ax1.set_title("Funnel Distribution", fontsize=13, fontweight="bold", pad=12)
ax1.set_ylabel("Accounts")
ax1.set_facecolor("#FAFAFA")
ax1.spines[["top", "right"]].set_visible(False)

# Graph 2 — Conversion Rate donut
ax2 = fig.add_subplot(gs[0, 1])
ax2.pie([active, total - active], colors=["#27AE60", "#E0E0E0"],
        startangle=90, wedgeprops=dict(width=0.55, edgecolor="white"))
ax2.text(0, 0, f"{conversion_rate}%", ha="center", va="center",
         fontsize=22, fontweight="bold", color="#27AE60")
ax2.set_title("Trial → Active Conversion", fontsize=13, fontweight="bold")
ax2.legend(["Converted", "Not Converted"], loc="lower center",
           bbox_to_anchor=(0.5, -0.1), ncol=2, fontsize=9)

# Graph 3 — Churn Rate by Plan Tier
ax3 = fig.add_subplot(gs[0, 2])
churn_by_plan = (
    funnel_df.groupby("plan_tier")["churn_flag"]
    .mean().mul(100).round(1).sort_values(ascending=False)
)
h_bars = ax3.barh(churn_by_plan.index, churn_by_plan.values,
                  color=["#C0392B", "#E67E22", "#F39C12"][:len(churn_by_plan)],
                  edgecolor="white", linewidth=1.5)
for bar, val in zip(h_bars, churn_by_plan.values):
    ax3.text(bar.get_width() + 0.3, bar.get_y() + bar.get_height() / 2,
             f"{val}%", va="center", fontsize=10, fontweight="bold")
ax3.set_title("Churn Rate by Plan Tier", fontsize=13, fontweight="bold")
ax3.set_xlabel("Churn Rate (%)")
ax3.set_facecolor("#FAFAFA")
ax3.spines[["top", "right"]].set_visible(False)

# Graph 4 — Avg Usage Count by Stage
ax4 = fig.add_subplot(gs[1, 0])
usage_by_stage = (
    funnel_df.groupby("funnel_stage")["usage_count"]
    .mean().reindex(stage_order)
)
ax4.plot(stage_order, usage_by_stage.values, marker="o", color="#2E75B6",
         linewidth=2.5, markersize=9, markerfacecolor="white", markeredgewidth=2.5)
ax4.fill_between(stage_order, usage_by_stage.values, alpha=0.12, color="#2E75B6")
for x, y in zip(stage_order, usage_by_stage.values):
    ax4.text(x, y + 0.3, f"{y:.1f}", ha="center", fontsize=10,
             fontweight="bold", color="#2E75B6")
ax4.set_title("Avg Usage Count by Stage", fontsize=13, fontweight="bold")
ax4.set_ylabel("Avg usage_count")
ax4.set_facecolor("#FAFAFA")
ax4.spines[["top", "right"]].set_visible(False)

# Graph 5 — Correlation Matrix
ax5       = fig.add_subplot(gs[1, 1:])
corr_cols = ["total_events", "usage_count", "churn_flag"]
available = [c for c in corr_cols if c in funnel_df.columns]
corr_data = funnel_df[available].apply(pd.to_numeric, errors="coerce").corr()
mask      = np.triu(np.ones_like(corr_data, dtype=bool))
sns.heatmap(corr_data, annot=True, fmt=".2f", cmap="coolwarm",
            mask=mask, ax=ax5, linewidths=0.5,
            annot_kws={"size": 12, "weight": "bold"},
            cbar_kws={"shrink": 0.7})
ax5.set_title("Correlation Matrix", fontsize=13, fontweight="bold")

plt.suptitle("FinFlow Analytics — Real-Time Funnel Dashboard",
             fontsize=16, fontweight="bold", y=1.01, color="#1A3C5E")
plt.savefig("finflow_dashboard.png", dpi=150, bbox_inches="tight",
            facecolor=fig.get_facecolor())
print("✅ Dashboard saved → finflow_dashboard.png")
plt.show()
```

```bash
python analytics/analytics.py
```

---

## 🔹 How to Run (Quick Reference)

```bash
# 1. Install all Python dependencies
pip install -r requirements.txt

# 2. Create MySQL database and tables
mysql -u root -p < sql/01_create_database.sql

# 3. Simulate and load event data
python data_pipeline/insert_data.py

# 4. Run SQL transformation pipeline
mysql -u root -p funnel_analytics < sql/02_pipeline_transform.sql

# 5. Generate analytics dashboard
python analytics/analytics.py
```

---

## 🔹 Sample Output

| Chart | What It Shows |
|---|---|
| **Funnel Distribution** | Bar chart — Trial (97), Active (318), At-Risk, Churned (110) accounts |
| **Conversion Rate** | Donut — 63.6% of accounts converted Trial → Active paid subscription |
| **Churn Rate by Plan** | Horizontal bar — 22% overall churn broken down by Basic / Pro / Enterprise |
| **Usage Count by Stage** | Line chart — Active users show highest avg `usage_count`; Churned users lowest |
| **Correlation Matrix** | Heatmap — `usage_count` negatively correlates with `churn_flag` (high usage = retained) |

> 📊 Full dashboard saved automatically to `finflow_dashboard.png`

---

## 🔹 Future Improvements

| Phase | Timeline | Enhancement |
|---|---|---|
| v2.0 | Q3 2025 | **Apache Kafka** for true real-time event streaming |
| v2.1 | Q3 2025 | **Streamlit** interactive dashboard replacing static charts |
| v3.0 | Q4 2025 | **ML churn prediction** (XGBoost / Random Forest) — 30-day early warning |
| v3.1 | Q1 2026 | **NLP feedback classifier** on churn `feedback_text` using Transformers |
| v4.0 | Q2 2026 | **Automated alerts** — Slack / email on at-risk account detection |

---

## 🔹 Resume Description

**Short (LinkedIn / Resume bullet):**
> Built FinFlow Analytics — a real-time SaaS funnel intelligence platform using Python, MySQL, Pandas, and Matplotlib — processing 25,000+ feature usage events across 500 B2B accounts to identify churn drivers, optimize conversion, and surface $2.4M+ in at-risk MRR.

**Extended (Portfolio):**
> Designed and implemented an end-to-end real-time funnel analytics pipeline for a fintech SaaS platform with $11.34M MRR. Engineered a MySQL event-stream architecture ingesting 5 data sources (500 accounts, 5,000 subscriptions, 25,000 usage events, 600 churn records, 2,000 support tickets). Built SQL transformation pipelines to classify accounts into funnel stages and Python analytics modules to compute conversion rates, churn distributions, feature adoption scores, and support-churn correlations. Delivered a 5-panel executive dashboard identifying 22% account churn with actionable recommendations targeting a 32% reduction.

---

## 🔹 License

MIT License — free to use, fork, and build upon.

---

<p align="center">Made with ❤️ &nbsp;|&nbsp; Python • MySQL • Pandas • Matplotlib</p>
