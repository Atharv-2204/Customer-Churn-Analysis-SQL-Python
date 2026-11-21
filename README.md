#  Customer Churn Analysis — SQL Database to Business Insights

An end-to-end churn analysis built in Python, extracting data from a relational SQLite database, resolving a real data quality bug, and calculating 13+ business KPIs to diagnose why customers leave and when.

---

##  Project Objective

To move beyond a single headline "churn rate" number and answer the questions a retention team actually needs answered:
- **How much** revenue is at risk from churned customers?
- **Which** customer segments (plan type, tenure, support history) churn most?
- **When** in a customer's lifecycle are they most likely to leave?
- **Why** do customers cancel — and does contacting support actually predict churn?

---

##  Dataset

**Source:** SQLite relational database (`customer_churn.db`) containing 3 linked tables

| Table | Contents |
|---|---|
| `db_customer` | Customer demographics — name, country, state, gender, date of birth, interests |
| `db_subscription` | Subscription lifecycle — plan type, contract type, dates, monthly charges, churn score, cancellation reason |
| `db_support` | Support interaction history — complaint dates, escalation flag, CSAT score |

Unlike a pre-merged single CSV, this project required extracting and joining **three separate relational tables** — a closer analogue to how real production data is structured.

---

##  Key Findings

1. **28.84% of customers have churned**, against a 71.16% retention rate — establishing the baseline scale of the problem.
2. **Basic plan customers churn at the highest rate (30.97%)**, notably higher than Premium (27.61%), suggesting price-sensitive customers on entry-level plans are the highest-risk segment.
3. **$28,449.77 in monthly recurring revenue is at risk** from already-churned customers — quantifying the direct financial cost of churn.
4. **Escalations do not meaningfully predict churn** (correlation ≈ -0.01, verified on the 1,756 customers who actually contacted support) — a counter-intuitive finding suggesting that customers who churn are not necessarily the ones who complain first. This points toward silent dissatisfaction or price/fit issues as more likely churn drivers than service friction.
5. **Churn timing analysis by tenure bucket** reveals whether customers leave early (onboarding failure) or late (long-term dissatisfaction) — see the tenure chart in the notebook for the full breakdown.
6. **Top cancellation reasons** were extracted directly from churned customers' stated reasons, turning the "how many churned" number into an actionable "why they churned" narrative.

---

##  Skills & Techniques Demonstrated

| Category | Techniques used |
|---|---|
| Data extraction | SQLite connection via `sqlite3`, dynamic table discovery, `PRAGMA table_info()` for schema inspection |
| Data cleaning | Column renaming, missing value handling, data type correction, standardization across 3 tables |
| Data quality debugging | Identified and resolved a duplicate-record bug (support table had 2,247 rows but only 1,758 unique customers) using sort + deduplication logic |
| Feature engineering | Churn flag creation, complaint count aggregation, tenure calculation (day-level), churn risk bucketing from churn score |
| Business metrics | Churn rate, retention rate, ARPU, revenue at risk, escalation rate, average complaints per user |
| Statistical rigor | Built and compared **incorrect (alphabetical) vs. correct (business-priority ordered)** categorical encoding to demonstrate the impact of encoding choice on correlation analysis |
| Data visualization | Seaborn heatmaps, pairplots, catplots/FacetGrid, time-series trend charts, pivot tables |

---

##  Sample Finding — Ordinal Encoding Matters

This project deliberately demonstrates a subtle but important statistics concept: **default alphabetical encoding of categorical variables can distort correlation analysis.**

```python
# Correct method — encode by actual business priority, not alphabetical order
order_mapping = {
    'plan_type': ['Basic', 'Standard', 'Premium'],
    'contract_type': ['Monthly', 'Annual'],
    'churn_risk': ['low', 'med', 'high'],
    'escalations': ['N', 'Y'],
}

for col, order in order_mapping.items():
    df_encoded[col] = pd.Categorical(
        df_encoded[col], categories=order, ordered=True
    ).codes
```

Both the incorrect and correct encoding heatmaps are included side-by-side in the notebook for direct comparison.

---

##  How to Run This Project

1. Clone this repository
2. Install dependencies: `pip install pandas numpy matplotlib seaborn`
3. Ensure `customer_churn.db` is in the project directory
4. Open `churn_analysis.ipynb` in Jupyter Notebook or JupyterLab
5. Run all cells in order

---

##  What I Learned

- How to extract and join data from a relational database rather than working from a single pre-merged file — including dynamically discovering table names and schemas.
- How a subtle merge issue (duplicate records in a support ticket log) can silently inflate row counts after a join, and how to catch and resolve it with a documented sort-and-deduplicate approach.
- Why encoding categorical variables alphabetically (versus by genuine business priority) can distort correlation results — and how to build both versions side-by-side to demonstrate the difference concretely.
- The importance of validating a surprising result before accepting it: an initial near-zero escalation-churn correlation was investigated for a possible data bias (NaN handling), and confirmed as a genuine finding rather than an artifact once the bias was ruled out.
- How to extend a churn rate number into a business narrative — tenure-based churn timing and cancellation reason breakdowns turn "what happened" into "why it happened and when."

---

##  About

Built by **[Atharv Dhole]** as part of a data analyst portfolio project.

- 🔗 LinkedIn: https://www.linkedin.com/in/atharv-dhole/
- 📧 Email: atharvdhole22@gmail.com

---
