# Marketplace Settlement & Bank Reconciliation Case Study

A real-world business problem I worked with involved reconciling e-commerce order data, marketplace settlements, and actual bank deposits.

The exact production systems and field names used by the company are not included here. The SQL, table names, and sample values below are simplified examples of the workflow.

## Business Problem

The finance team regularly needed to perform bank reconciliation across several data sources.

For Amazon transactions, three numbers did not always agree:

1. Sales and transaction data stored in the internal database
2. Settlement amounts reported by Amazon
3. Deposits actually received in the company bank account

Finance primarily worked in Excel and NetSuite, while operational and transaction-level data was available through the internal database.

A simple comparison of total sales against the bank deposit was not enough because marketplace settlements could include:

* Marketplace fees
* Refunds
* Returns
* Advertising or service charges
* Shipping-related adjustments
* Chargebacks
* Timing differences
* Prior-period adjustments

The challenge was not just finding a different number. It was explaining **why the numbers were different** and determining whether the difference was expected or required investigation.

---

## Initial Workflow

The original process depended heavily on manual work.

Finance would:

* Download marketplace settlement reports
* Export transaction data
* Review bank deposits
* Compare spreadsheets
* Search for unmatched amounts
* Investigate discrepancies manually
* Enter or reconcile results in NetSuite

This became increasingly difficult as transaction volume increased and additional marketplaces were added.

---

## My Role

I supported the reconciliation process from the data and systems side.

My work included:

* Pulling transaction-level data using SQL
* Using Python to clean and combine settlement files
* Helping finance compare marketplace activity with internal records
* Identifying common causes of unmatched transactions
* Building repeatable reconciliation logic
* Reducing the amount of manual record-by-record checking

Finance continued to own the final accounting and NetSuite reconciliation process.

My goal was to make the data reaching them easier to explain and easier to match.

---

## Step 1: Establish the Reconciliation Layers

Instead of comparing one large total, I separated the problem into layers.

### Layer 1 — Internal Transactions

Typical fields included:

* order ID
* marketplace
* transaction date
* gross sales
* tax
* shipping revenue
* refund amount
* order status

### Layer 2 — Marketplace Settlement

Settlement files could contain:

* order payments
* selling fees
* fulfillment fees
* refunds
* chargebacks
* adjustments
* other marketplace charges

### Layer 3 — Bank Deposit

The final settlement should eventually correspond to a deposit received by the company.

Because marketplace activity and deposits were not always recorded on the same date, settlement ID and settlement period were often more useful than simply matching by transaction date.

---

## Step 2: Pull Internal Transactions

A simplified query could look like:

```sql
SELECT
    order_id,
    marketplace,
    transaction_date,
    gross_amount,
    refund_amount,
    order_status
FROM marketplace_orders
WHERE marketplace = 'Amazon'
  AND transaction_date >= :period_start
  AND transaction_date < :period_end;
```

The actual production logic depended on the available source tables and business rules.

---

## Step 3: Normalize Marketplace Settlement Data

Marketplace exports were not always structured in the same way as internal order data.

Python was useful for:

* standardizing column names
* converting dates and currency values
* separating transaction types
* handling missing values
* aggregating settlement components
* preparing data for comparison

A simplified example:

```python
import pandas as pd

settlement = pd.read_csv("amazon_settlement.csv")

settlement.columns = (
    settlement.columns
    .str.strip()
    .str.lower()
    .str.replace(" ", "_")
)

settlement["amount"] = pd.to_numeric(
    settlement["amount"],
    errors="coerce"
).fillna(0)

summary = (
    settlement
    .groupby("transaction_type")["amount"]
    .sum()
    .sort_values()
)

print(summary)
```

This made it easier to see how gross marketplace activity was reduced or adjusted before reaching the bank.

---

## Step 4: Build a Settlement Bridge

The most useful output was not simply:

> Database = X
> Bank = Y
> Difference = Z

Instead, I tried to explain the movement between those values.

A simplified reconciliation might look like:

| Component                     |   Amount |
| ----------------------------- | -------: |
| Gross order activity          | $128,450 |
| Refunds                       |  -$6,820 |
| Marketplace fees              | -$18,340 |
| Fulfillment / service charges |  -$4,910 |
| Other adjustments             |    +$620 |
| Expected settlement           |  $99,000 |
| Bank deposit                  |  $99,000 |
| Unexplained difference        |       $0 |

The values above are illustrative only.

The important part of the workflow was creating an explainable bridge between operational sales data and the final settlement.

---

## Step 5: Investigate Unmatched Differences

When the reconciliation still had a difference, I would narrow the investigation rather than manually reviewing every transaction.

Common checks included:

### Timing Differences

A transaction may appear in one reporting period but settle in the next.

```sql
SELECT
    order_id,
    transaction_date,
    settlement_id,
    settlement_date,
    amount
FROM marketplace_transactions
WHERE settlement_date <> transaction_date;
```

### Refunds

Refunds may be issued after the original sale and appear in a later settlement.

### Duplicate or Missing Records

```sql
SELECT
    external_transaction_id,
    COUNT(*) AS record_count
FROM marketplace_transactions
GROUP BY external_transaction_id
HAVING COUNT(*) > 1;
```

### Unmapped Transaction Types

Marketplace files occasionally introduced transaction descriptions or adjustment types that were not included in existing reconciliation rules.

I would identify those separately instead of silently forcing them into an existing category.

---

## Step 6: Flag Exceptions Instead of Reviewing Everything

As the process became more repeatable, the better approach was to focus finance attention on exceptions.

For example:

```sql
SELECT
    settlement_id,
    expected_amount,
    bank_amount,
    ABS(expected_amount - bank_amount) AS difference
FROM reconciliation_summary
WHERE ABS(expected_amount - bank_amount) > :tolerance;
```

A tolerance could be defined based on accounting policy rather than assuming every small difference represented a problem.

The goal was to move from:

**review every transaction**

to:

**review transactions or settlements that actually need attention**

---

## Step 7: Finance Review

The resulting reconciliation output could then be shared with Finance for Excel and NetSuite processing.

Instead of receiving raw transaction data, Finance could work with:

* settlement totals
* categorized adjustments
* matched deposits
* unresolved differences
* records requiring manual review

This created a clearer handoff between operational data and accounting.

---

## Automation Over Time

The reconciliation process did not become fully automated immediately.

It evolved gradually:

**Manual exports and spreadsheet matching**

↓

**SQL-based data extraction**

↓

**Python-assisted cleanup and normalization**

↓

**Repeatable reconciliation logic**

↓

**Exception-focused review**

This incremental approach was practical because finance still needed visibility into how each number was calculated.

Automation reduced repetitive work without turning the accounting process into a black box.

---

## Where AI Could Help Today

If I were improving the same workflow today, I would use AI selectively for tasks such as:

* categorizing unfamiliar transaction descriptions
* summarizing unresolved reconciliation exceptions
* drafting investigation notes
* generating SQL starting points
* identifying missing validation checks
* converting technical findings into finance-facing explanations

I would not allow AI to determine the final accounting treatment or confirm a reconciliation without validating the actual transaction and system-of-record data.

---

## Business Value

The main value of this work was not a more complicated technical stack.

It was making a messy cross-system process easier to operate.

The workflow helped:

* reduce repetitive manual matching
* make settlement differences easier to explain
* identify exceptions faster
* improve communication between data and finance
* create more consistent reconciliation logic
* support a more scalable process as marketplace activity increased

---

## What This Case Demonstrates

This type of problem sits between several functions:

**E-commerce Operations → Data → Business Systems → Finance**

Solving it required more than SQL alone.

It required understanding what each system represented, how money moved through the business, where timing and fee differences occurred, and what Finance actually needed to complete the reconciliation.

That combination of technical analysis and business-process understanding is the type of work I enjoy most.
