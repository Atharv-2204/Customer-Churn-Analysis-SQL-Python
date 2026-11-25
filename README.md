# Customer Churn Analysis: From a SQLite Database to Retention Insights

A Python analysis of customer churn for a subscription service. The data comes from a three-table SQLite database of 5,250 customers. I cleaned and merged it in a Jupyter notebook, calculated seven KPIs, broke churn down by plan, state and cancellation reason, and tested whether support escalations relate to churn.

## Contents

- [Questions](#questions)
- [Dataset](#dataset)
- [Method](#method)
- [Results](#results)
- [Encoding for the correlation heatmap](#encoding-for-the-correlation-heatmap)
- [Notes and limitations](#notes-and-limitations)
- [Project files](#project-files)
- [How to run](#how-to-run)
- [What I learned](#what-i-learned)
- [Contact](#contact)

## Questions

- How many customers churn, and how much monthly revenue goes with them?
- Which plans and states churn the most?
- Why do customers cancel?
- Do support escalations relate to churn?

## Dataset

The source is `customer_churn.db`, a SQLite database with three linked tables. The customers are in eight Indian states.

| Table | Rows | Columns |
|---|---|---|
| `db_customer` | 5,250 | customerid, name, country, state, gender, dob, interests, pincode |
| `db_subscription` | 5,250 | customerid, subscription_start_date, subscription_type, renewal_date, plan_type, contract_type, cancellation_date, cancellation_reason, monthly_charges, cltv, churn_score |
| `db_support` | 2,238 | customerid, complaint_date, escalations, csat_score, col_1, comment |

## Method

The notebook lists the tables and checks their schemas with `PRAGMA table_info()`, loads all three into pandas, cleans them and merges them into one table with one row per customer.

Cleaning:
- `db_customer`: renamed `name` to `customer_name`, dropped `interests` (946 of 5,250 filled) and `pincode` (empty), converted `dob` to a date, standardized gender (Men and Women became Male and Female), and filled the 734 missing countries from each customer's state.
- `db_subscription`: converted the three date columns to dates and added a `churn_flag`, which is 1 when a cancellation date exists.
- `db_support`: dropped `col_1` (empty) and `comment`, and converted `complaint_date` to a date.

### Duplicate support records

`db_support` has 2,238 rows but only 1,756 distinct customers. Merging it as it was turned 5,250 customers into 5,732 rows. I counted each customer's complaints, then kept only the most recent complaint per customer:

```python
df_db_support['complaint_count'] = df_db_support.groupby('customerid')['customerid'].transform('count')
df_db_support = df_db_support.sort_values('complaint_date').drop_duplicates('customerid', keep='last')
```

After the fix the merged table has 5,250 rows again. One effect of this step: the escalation flag used later is the one from each customer's latest complaint.

## Results

| Metric | Value |
|---|---|
| Customers | 5,250 |
| Churned | 1,514 (28.84%) |
| Retention rate | 71.16% |
| ARPU (average monthly charge) | 18.73 |
| Revenue at risk | $28,449.77 per month |
| Average tenure | 1,463 days |
| Escalation rate | 18.72% of all customers |
| Average complaints per customer | 0.43 |

Findings:

1. The monthly charges of churned customers add up to $28,449.77, about 29% of the 98,317 charged across all customers.
2. Basic has the highest plan churn at 30.97% (1,240 customers), then Standard at 28.62% (2,257) and Premium at 27.61% (1,753). The spread is about 3 points. Basic accounts for 14% of monthly charges and Premium for 55%.
3. Meghalaya has the highest state churn at 31.70% (814 customers), followed by Maharashtra at 30.53% (855). Rajasthan is the lowest at 27.08% (554). Delhi has the most customers, 1,077, at 27.76%.
4. Of the 1,514 customers who cancelled, 526 (34.7%) switched to a competitor. Forgetting to cancel a trial accounts for 260 (17.2%), not enough content for 252 (16.6%), price for 241 (15.9%) and streaming quality for 235 (15.5%).
5. Escalations show no relationship with churn. The correlation is -0.01 across the 1,756 customers who contacted support, 983 of whom (56%) had an escalation. Those 1,756 customers are a third of the 5,250.
6. The number of customers who churn each month rises from 2019 to a peak of about 40 in mid 2024, then drops sharply from late 2024.

## Encoding for the correlation heatmap

Plan type, contract type, churn risk and escalations need numeric codes before they can go into a correlation heatmap. Pandas assigns codes alphabetically, which puts Premium ahead of Standard and breaks the Basic, Standard, Premium order. The notebook draws the heatmap twice, once with the default codes and once with codes that follow the business order:

```python
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

`churn_risk` is built from `churn_score`: below 50 is low, 50 to 69 is med and 70 or above is high.

## Notes and limitations

- The chart of churn rate by tenure bucket shows 100% for every bucket under 365 days. This comes from how tenure is defined. Active customers are measured up to the day the notebook runs, so all of them land in the 365+ bucket, and only customers who cancelled can appear in the shorter buckets. The chart does not show when customers leave, so it is left out of the findings. The average tenure of 1,463 days also depends on the run date.
- Escalation status comes from each customer's latest complaint only, as described in the Method section.
- The differences between plans and between states are a few percentage points. I did not test whether they are statistically significant.

## Project files

```
├── churn_analysis.ipynb      # full analysis
├── customer_churn.db         # source database
└── exported_churn_data.csv   # merged table, written when the notebook runs
```

## How to run

1. Clone this repository.
2. Install the dependencies: `pip install pandas numpy matplotlib seaborn`
3. Keep `customer_churn.db` in the same folder as the notebook.
4. Open `churn_analysis.ipynb` in Jupyter Notebook or JupyterLab.
5. Run all cells in order.

## What I learned

- Reading related tables from a database and joining them in pandas. A support table with several rows per customer quietly multiplied the merged table from 5,250 to 5,732 rows, and I only noticed because the row count did not match the customer count.
- An alphabetical numeric code can distort a correlation. Building the heatmap with both encodings made the difference easy to see.
- A near-zero correlation needs a check before it is reported. I made sure customers who never contacted support were left out of the escalation correlation instead of being counted as "no escalation".
- A list of cancellation reasons says more about why customers leave than any plan or state breakdown. Price is the reason for 15.9% of cancellations, and switching to a competitor for 34.7%.

## Contact

Atharv Dhole

- LinkedIn: https://www.linkedin.com/in/atharv-dhole/
- Email: atharvdhole22@gmail.com
