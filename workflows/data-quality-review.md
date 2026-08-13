# Data Quality Review

A practical checklist for reviewing operational data before it is used for reporting, analysis, or business decisions.

I use this type of workflow when a report looks wrong, two systems disagree, or a dataset needs to be validated before it becomes part of a recurring process.

## What I Check First

Before looking for complex causes, I usually start with a few basic questions:

* Are records missing?
* Are there duplicates?
* Are important fields null?
* Are statuses valid?
* Are timestamps reasonable?
* Do related systems agree?
* Are there unexpected outliers?
* Are old records still being treated as active?

---

## Prompt Template

```text
I am reviewing a dataset for potential data-quality issues.

Business context:
[Describe what the data is used for]

Dataset:
[Orders / inventory / shipping / payments / customer data / etc.]

Important fields:
- [field 1]
- [field 2]
- [field 3]

Known business rules:
- [rule 1]
- [rule 2]

Known issue:
[Describe the problem, if there is one]

Please help me organize a data-quality review.

Check for:
1. Missing values
2. Duplicate records
3. Invalid values or statuses
4. Cross-field inconsistencies
5. Cross-system mismatches
6. Outliers
7. Stale records
8. Business-rule violations

For each issue type:
- Explain what to check
- Suggest a SQL or Excel validation approach
- Explain possible business impact
- Flag any assumption that requires confirmation

Do not assume that an unusual value is automatically incorrect.
```

---

## Example: Marketplace Order Data

Assume an order table contains:

* order_id
* marketplace
* sku
* order_status
* quantity
* order_amount
* shipping_cost
* created_at
* updated_at

### Duplicate Orders

```sql
SELECT
    order_id,
    COUNT(*) AS record_count
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

Duplicate order records can inflate sales, order volume, inventory usage, and shipping metrics.

---

### Missing Required Fields

```sql
SELECT *
FROM orders
WHERE order_id IS NULL
   OR marketplace IS NULL
   OR sku IS NULL
   OR order_status IS NULL;
```

Not every null value is necessarily a data problem, so required fields should be confirmed with the business rules first.

---

### Invalid Status Values

```sql
SELECT DISTINCT order_status
FROM orders;
```

If the approved statuses are:

* Pending
* Processing
* Shipped
* Delivered
* Cancelled

then unexpected values should be reviewed before they are included in reporting logic.

---

### Negative or Unusual Values

```sql
SELECT *
FROM orders
WHERE quantity <= 0
   OR order_amount < 0
   OR shipping_cost < 0;
```

Negative values may indicate an error, but they may also represent returns, refunds, or adjustments.

The business meaning should be confirmed before treating them as bad data.

---

### Stale Orders

```sql
SELECT *
FROM orders
WHERE order_status IN ('Pending', 'Processing')
  AND updated_at < CURRENT_TIMESTAMP - INTERVAL '3 days';
```

Older active-status records may indicate:

* Failed workflow updates
* Delayed fulfillment
* Missing status synchronization
* Legitimate operational delays

These records should be reviewed based on the normal processing SLA.

---

## Example: Cross-System Inventory Check

If inventory exists in both an internal system and a marketplace feed, I would compare the systems instead of reviewing each one separately.

```sql
SELECT
    i.sku,
    i.available_qty AS internal_qty,
    m.available_qty AS marketplace_qty,
    i.available_qty - m.available_qty AS difference
FROM internal_inventory i
JOIN marketplace_inventory m
    ON i.sku = m.sku
WHERE i.available_qty <> m.available_qty;
```

A mismatch does not automatically identify which system is wrong.

The next checks may include:

* Last sync time
* Warehouse source
* Recent inventory adjustments
* Failed integration jobs
* SKU mapping
* Reserved vs. available stock logic

---

## Business Impact

Data-quality problems often look technical but can quickly become business problems.

For example:

**Duplicate orders**

* Inflated revenue reporting
* Incorrect inventory reduction
* Incorrect KPI calculations

**Stale inventory**

* Overselling
* Cancelled orders
* Poor customer experience

**Missing shipping cost**

* Incorrect margin reporting
* Incomplete logistics analysis

**Invalid status**

* Orders disappearing from reports
* Incorrect operational queues
* Broken downstream automation

---

## Using AI in the Review

I use AI mainly to help:

* organize validation checks
* identify checks I may have missed
* draft SQL starting points
* summarize patterns
* explain findings to non-technical stakeholders

I would still validate the final result using the actual dataset, system records, and business rules.

AI can suggest that something looks unusual. It cannot decide whether the value is truly wrong without the business context.

---

## Final Review Checklist

Before I consider a dataset ready for business use, I would confirm:

* Key records are complete
* Required fields are populated
* Duplicate logic is understood
* Status values are valid
* Important numerical values are reasonable
* Timestamps follow expected workflow behavior
* Cross-system differences are explained
* Known exceptions are documented
* Business rules match the reporting logic
* Any AI-generated observations have been independently checked
