# E-Commerce RFM Segmentation & Q4 Budget Optimization
### Case Study — Protecting Margin on a 540K+ Transaction Dataset with a Native MySQL ELT Pipeline

---

## 1. The Business Challenge

Heading into Q4, the marketing team planned to send a 30% discount code to the entire customer database to hit revenue targets. Finance flagged that this "spray and pray" approach would quietly destroy profit margins in two ways:

1. **Subsidizing loyal customers** who would have paid full price anyway.
2. **Wasting budget on "one-and-done" bargain hunters** who are unlikely to convert into repeat, full-price buyers.

**Objective:** Replace the blanket discount with a data-driven targeting model that identifies exactly which customer segments should receive the marketing budget — maximizing reactivation revenue while cutting ad spend to zero for dead cohorts.

---

## 2. My Approach

I built an end-to-end **ELT (Extract, Load, Transform) pipeline natively in MySQL** — no external scripting layer — to process the **UCI Online Retail Dataset** (540,000+ rows) and score every customer on RFM (Recency, Frequency, Monetary) behavior.

The pipeline runs in three phases:

1. **Clean & Load** — ingest the raw transaction log, strip invalid rows, and cast types correctly.
2. **Score** — calculate each customer's Recency, Frequency, and Monetary value, then bucket them into statistical quantiles.
3. **Segment & Recommend** — translate numeric scores into business personas and a Q4 spending plan.

---

## 3. Technical Deep-Dive

| Technique | How it was used |
|---|---|
| **Data Cleaning & Type Casting** | Filtered out 135,000+ null "ghost" customer IDs, stripped negative quantities (returns/refunds), and cast text-based dates with `STR_TO_DATE()` |
| **Common Table Expressions (CTEs)** | Phased the logic from raw ingestion through to final segmentation, keeping each transformation step readable and testable |
| **Advanced Aggregation** | Calculated Frequency with `COUNT(DISTINCT InvoiceNo)` rather than line-item counts, avoiding false inflation of customer activity |
| **Window Functions** | Used `NTILE(5)` to distribute each behavioral variable into discrete 1–5 statistical quantiles |
| **Heuristic Logic** | Applied top-down `CASE WHEN` rules to map numeric cohort codes (e.g. `555`, `145`) into actionable personas |

---

## 4. The RFM Model

Every customer is scored from 1 (lowest) to 5 (highest) across three dimensions:

- **Recency (R):** Days since the customer's last purchase
- **Frequency (F):** Total number of distinct checkout events
- **Monetary (M):** Total lifetime revenue generated

---

## 5. Results: Three Q4 Budget Plays

| Play | Target Segment | Discount | Why |
|---|---|---|---|
| **Reactivation** | `At-Risk` / `Can't Lose Them` | 30% | High historical LTV but declining recency — the discount buys back a profitable recurring revenue stream |
| **Margin Protection** | `Champions` (highest R, F, M) | 0% | Already buying frequently at full price — discounting destroys guaranteed margin; offer zero-cost perks (e.g. early access) instead |
| **Churn** | `Hibernating` | 0% | Low-value, single-purchase bargain hunters — not worth acquisition spend |

**Net effect:** the marketing budget is redirected away from a blanket blast and toward the two segments where it actually changes behavior, while cutting spend entirely on dead cohorts.

---

## 6. Tech Stack

MySQL 8.0+ · SQL (CTEs, window functions, heuristic `CASE WHEN` logic)

---

## 7. How to Run

**Step 1 — Database Setup**
Ensure MySQL Server 8.0+ is installed, then create the landing table:

```sql
CREATE TABLE raw_online_retail (
    InvoiceNo VARCHAR(20),
    StockCode VARCHAR(20),
    Description VARCHAR(255),
    Quantity INT,
    InvoiceDate VARCHAR(50),
    UnitPrice DECIMAL(10, 4),
    CustomerID VARCHAR(20),
    Country VARCHAR(50)
);
```

**Step 2 — Data Ingestion**
Place the `Online Retail.csv` file in your MySQL `Uploads` directory (defined by your `secure_file_priv` variable) and run:

```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Online Retail.csv'
INTO TABLE raw_online_retail
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

**Step 3 — Execute Pipeline**
Run `rfm_segmentation_pipeline.sql` to clean the data, calculate the metrics, apply the window functions, and output the final segmented business matrix.
