# ⚙️ Setup Guide — FinFlow Analytics
### Complete Step-by-Step Installation & Configuration

> **Stack:** Python 3.9+ | MySQL 8.0+ | Pandas | Matplotlib | mysql-connector-python
> **Est. Setup Time:** ~10 minutes

---

## 📋 Prerequisites

Before you begin, ensure the following are installed and running:

| Requirement | Version | Check Command |
|---|---|---|
| Python | 3.9+ | `python --version` |
| MySQL Server | 8.0+ | `mysql --version` |
| pip | Latest | `pip --version` |

Install Python dependencies first:

```bash
pip install -r requirements.txt
```

---

## 📌 Step 1 — MySQL Database Setup

Open **MySQL Workbench** or connect via terminal:

```bash
mysql -u root -p
```

Then run the following SQL to create the database and tables:

```sql
-- ─────────────────────────────────────────────────────────
-- CREATE DATABASE
-- ─────────────────────────────────────────────────────────
CREATE DATABASE IF NOT EXISTS funnel_analytics;
USE funnel_analytics;

-- ─────────────────────────────────────────────────────────
-- RESET (safe to re-run)
-- ─────────────────────────────────────────────────────────
DROP TABLE IF EXISTS event_stream;
DROP TABLE IF EXISTS realtime_funnel;

-- ─────────────────────────────────────────────────────────
-- TABLE 1: event_stream  (raw event ingest layer)
-- ─────────────────────────────────────────────────────────
CREATE TABLE event_stream (
    event_id            INT AUTO_INCREMENT PRIMARY KEY,
    account_id          INT,
    event_type          VARCHAR(50),              -- signup | subscribe | usage | churn
    feature_usage_count INT          DEFAULT 0,
    event_time          TIMESTAMP    DEFAULT CURRENT_TIMESTAMP
);

-- ─────────────────────────────────────────────────────────
-- TABLE 2: realtime_funnel  (aggregated stage view)
-- ─────────────────────────────────────────────────────────
CREATE TABLE realtime_funnel (
    account_id  INT     PRIMARY KEY,
    signed_up   TINYINT DEFAULT 0,
    subscribed  TINYINT DEFAULT 0,
    usage_count INT     DEFAULT 0,
    churned     TINYINT DEFAULT 0
);
```

✅ **Expected output:** `Query OK` for each statement — database and both tables created.

---

## 📌 Step 2 — Insert Simulated Event Data (Python)

Save as **`data_pipeline/insert_data.py`** and run it.

This script generates **1,000 random events** across 200 simulated accounts
covering all four event types: `signup`, `subscribe`, `usage`, `churn`.

```python
# data_pipeline/insert_data.py

import mysql.connector
import random

# ── DB Connection ──────────────────────────────────────────
cnx = mysql.connector.connect(
    host="127.0.0.1",
    user="root",
    password="YOUR_PASSWORD",     # ← replace with your MySQL password
    database="funnel_analytics"
)
cursor = cnx.cursor()

# ── Config ─────────────────────────────────────────────────
event_types = ['signup', 'subscribe', 'usage', 'churn']
NUM_EVENTS  = 1000
NUM_ACCOUNTS = 200

# ── Generate & Insert Events ───────────────────────────────
for _ in range(NUM_EVENTS):
    account_id = random.randint(1, NUM_ACCOUNTS)
    event      = random.choice(event_types)
    usage      = random.randint(1, 100) if event == 'usage' else 0

    cursor.execute("""
        INSERT INTO event_stream (account_id, event_type, feature_usage_count)
        VALUES (%s, %s, %s)
    """, (account_id, event, usage))

cnx.commit()
print(f"✅ Inserted {NUM_EVENTS} events into event_stream")

cursor.close()
cnx.close()
```

```bash
python data_pipeline/insert_data.py
```

✅ **Expected output:** `Inserted 1000 events into event_stream`

---

## 📌 Step 3 — Run Pipeline Transformation

Save as **`data_pipeline/pipeline.py`** and run it.

This script reads raw events from `event_stream`, aggregates them per account,
classifies funnel stage flags, and writes the result into `realtime_funnel`.

```python
# data_pipeline/pipeline.py

import mysql.connector

# ── DB Connection ──────────────────────────────────────────
cnx = mysql.connector.connect(
    host="127.0.0.1",
    user="root",
    password="YOUR_PASSWORD",     # ← replace with your MySQL password
    database="funnel_analytics"
)
cursor = cnx.cursor()

# ── Transformation Query ───────────────────────────────────
# REPLACE INTO upserts: inserts new rows or updates existing ones
cursor.execute("""
    REPLACE INTO realtime_funnel
    SELECT
        account_id,
        MAX(CASE WHEN event_type = 'signup'     THEN 1 ELSE 0 END) AS signed_up,
        MAX(CASE WHEN event_type = 'subscribe'  THEN 1 ELSE 0 END) AS subscribed,
        MAX(feature_usage_count)                                    AS usage_count,
        MAX(CASE WHEN event_type = 'churn'      THEN 1 ELSE 0 END) AS churned
    FROM  event_stream
    GROUP BY account_id
""")

cnx.commit()
print(f"✅ Pipeline complete — {cursor.rowcount} accounts written to realtime_funnel")

cursor.close()
cnx.close()
```

```bash
python data_pipeline/pipeline.py
```

✅ **Expected output:** `Pipeline complete — N accounts written to realtime_funnel`

---

## 📌 Step 4 — Run Analytics & Generate Charts

Save as **`analytics/analytics.py`** and run it.

This script loads `realtime_funnel`, classifies each account into a funnel stage,
computes key KPIs, and renders **4 charts**.

```python
# analytics/analytics.py

import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import mysql.connector

# ── DB Connection ──────────────────────────────────────────
cnx = mysql.connector.connect(
    host="127.0.0.1",
    user="root",
    password="YOUR_PASSWORD",     # ← replace with your MySQL password
    database="funnel_analytics"
)

df = pd.read_sql("SELECT * FROM realtime_funnel", cnx)
cnx.close()

# ── Stage Classification ───────────────────────────────────
def get_stage(row):
    if   row['churned']     == 1:    return 'Churned'
    elif row['usage_count'] >  50:   return 'Power User'
    elif row['usage_count'] >  0:    return 'Active'
    elif row['subscribed']  == 1:    return 'Subscribed'
    else:                            return 'Signup'

df['stage']     = df.apply(get_stage, axis=1)
df['converted'] = df['stage'].isin(['Subscribed', 'Active', 'Power User']).astype(int)

# ── KPI Summary ────────────────────────────────────────────
total           = len(df)
conversion_rate = df['converted'].mean()
churn_rate      = df['churned'].mean()

print(f"\n{'='*40}")
print(f"  FinFlow Analytics — KPI Summary")
print(f"{'='*40}")
print(f"  Total Accounts  : {total}")
print(f"  Conversion Rate : {conversion_rate:.1%}")
print(f"  Churn Rate      : {churn_rate:.1%}")
print(f"  Stage Breakdown :")
print(df['stage'].value_counts().to_string())
print(f"{'='*40}\n")

# ── Dashboard Setup ────────────────────────────────────────
fig = plt.figure(figsize=(16, 10))
fig.patch.set_facecolor("#F5F7FA")
gs  = gridspec.GridSpec(2, 2, figure=fig, hspace=0.42, wspace=0.32)

COLORS = ["#2E75B6", "#27AE60", "#F39C12", "#C0392B", "#8E44AD"]

# ── Chart 1: Funnel Distribution ───────────────────────────
ax1    = fig.add_subplot(gs[0, 0])
funnel = df['stage'].value_counts()
bars   = ax1.bar(funnel.index, funnel.values,
                 color=COLORS[:len(funnel)], edgecolor="white", linewidth=1.5)
for bar, val in zip(bars, funnel.values):
    ax1.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 0.4,
             str(val), ha="center", fontsize=10, fontweight="bold")
ax1.set_title("Funnel Distribution", fontsize=13, fontweight="bold", pad=10)
ax1.set_ylabel("Accounts")
ax1.set_facecolor("#FAFAFA")
ax1.spines[["top", "right"]].set_visible(False)

# ── Chart 2: Conversion Rate ───────────────────────────────
ax2    = fig.add_subplot(gs[0, 1])
labels = ['Converted', 'Not Converted']
values = [conversion_rate, 1 - conversion_rate]
bars2  = ax2.bar(labels, values,
                 color=["#27AE60", "#E0E0E0"], edgecolor="white", linewidth=1.5)
for bar, val in zip(bars2, values):
    ax2.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 0.01,
             f"{val:.1%}", ha="center", fontsize=11, fontweight="bold")
ax2.set_title("Conversion Rate", fontsize=13, fontweight="bold", pad=10)
ax2.set_ylabel("Rate")
ax2.set_ylim(0, 1.15)
ax2.set_facecolor("#FAFAFA")
ax2.spines[["top", "right"]].set_visible(False)

# ── Chart 3: Churn Rate ────────────────────────────────────
ax3    = fig.add_subplot(gs[1, 0])
labels = ['Churned', 'Retained']
values = [churn_rate, 1 - churn_rate]
bars3  = ax3.bar(labels, values,
                 color=["#C0392B", "#27AE60"], edgecolor="white", linewidth=1.5)
for bar, val in zip(bars3, values):
    ax3.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 0.01,
             f"{val:.1%}", ha="center", fontsize=11, fontweight="bold")
ax3.set_title("Churn Rate", fontsize=13, fontweight="bold", pad=10)
ax3.set_ylabel("Rate")
ax3.set_ylim(0, 1.15)
ax3.set_facecolor("#FAFAFA")
ax3.spines[["top", "right"]].set_visible(False)

# ── Chart 4: Avg Usage Count by Stage ─────────────────────
ax4           = fig.add_subplot(gs[1, 1])
usage_summary = df.groupby('stage')['usage_count'].mean().sort_values(ascending=False)
bars4         = ax4.bar(usage_summary.index, usage_summary.values,
                        color=COLORS[:len(usage_summary)], edgecolor="white", linewidth=1.5)
for bar, val in zip(bars4, usage_summary.values):
    ax4.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 0.3,
             f"{val:.1f}", ha="center", fontsize=10, fontweight="bold")
ax4.set_title("Avg Usage Count by Stage", fontsize=13, fontweight="bold", pad=10)
ax4.set_ylabel("Avg usage_count")
ax4.set_facecolor("#FAFAFA")
ax4.spines[["top", "right"]].set_visible(False)

# ── Save & Show ────────────────────────────────────────────
plt.suptitle("FinFlow Analytics — Funnel Intelligence Dashboard",
             fontsize=15, fontweight="bold", y=1.01, color="#1A3C5E")
plt.savefig("finflow_dashboard.png", dpi=150,
            bbox_inches="tight", facecolor=fig.get_facecolor())
print("✅ Dashboard saved → finflow_dashboard.png")
plt.show()
```

```bash
python analytics/analytics.py
```

✅ **Expected output:**
```
========================================
  FinFlow Analytics — KPI Summary
========================================
  Total Accounts  : 200
  Conversion Rate : 74.5%
  Churn Rate      : 25.0%
  Stage Breakdown :
  Active       82
  Churned      50
  Power User   41
  Subscribed   26
  Signup        1
========================================

✅ Dashboard saved → finflow_dashboard.png
```

---

## ▶️ How to Run — Quick Reference

Run all steps in order from your project root:

```bash
# Step 1 — Create database and tables (MySQL shell)
mysql -u root -p < sql/01_create_database.sql

# Step 2 — Generate and insert 1,000 simulated events
python data_pipeline/insert_data.py

# Step 3 — Transform events → realtime_funnel stage table
python data_pipeline/pipeline.py

# Step 4 — Generate analytics dashboard + save chart
python analytics/analytics.py
```

---

## 🗂️ Output Files

| File | Description |
|---|---|
| `finflow_dashboard.png` | 4-panel analytics chart (auto-saved on run) |
| `realtime_funnel` table | Aggregated account-level stage data in MySQL |

---

## 🛠️ Troubleshooting

| Error | Fix |
|---|---|
| `Access denied for user 'root'` | Check MySQL password in connection config |
| `Unknown database 'funnel_analytics'` | Re-run Step 1 SQL setup |
| `ModuleNotFoundError: No module named 'mysql'` | Run `pip install mysql-connector-python` |
| `ModuleNotFoundError: No module named 'pandas'` | Run `pip install pandas matplotlib seaborn` |
| `Table 'realtime_funnel' doesn't exist` | Re-run Step 1 SQL before Step 3 |
| Dashboard shows empty charts | Ensure Step 2 and Step 3 ran successfully first |

---

## 📁 File Reference

```
finflow-analytics/
│
├── sql/
│   └── 01_create_database.sql       ← Step 1 SQL (copy from above)
│
├── data_pipeline/
│   ├── insert_data.py               ← Step 2 script
│   └── pipeline.py                  ← Step 3 script
│
├── analytics/
│   └── analytics.py                 ← Step 4 script
│
├── Setup.md                         ← This file
├── README.md
├── requirements.txt
└── .gitignore
```

---

*FinFlow Analytics v1.0 | Setup Guide | Python • MySQL • Pandas • Matplotlib*
