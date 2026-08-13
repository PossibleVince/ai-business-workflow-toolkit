# Inventory Mismatch Case Study

A simple example showing how I would investigate an inventory mismatch between an internal system and an external marketplace.

This scenario is based on a common e-commerce operations problem: one system says inventory is available while another system shows zero stock.

## Situation

Operations reports that a product is still available for sale on a marketplace even though the warehouse has no physical inventory remaining.

Example:

* SKU: `SKU-123`
* Marketplace available quantity: 120
* Internal available quantity: 0
* Warehouse physical quantity: 0
* Several new orders were accepted after inventory reached zero

The immediate risk is overselling and additional order cancellations.

---

## Step 1: Confirm the Facts

Before trying to explain the issue, I would confirm the basic facts from each system.

Questions:

* Is the internal inventory quantity current?
* When did the marketplace quantity last update?
* Is the warehouse using the same SKU?
* Did any manual adjustment occur?
* Are reserved units being handled differently from available units?
* Did the synchronization process fail?

I would avoid assuming the marketplace is wrong until each source is checked.

---

## Step 2: Check Internal Inventory

Example query:

```sql
SELECT
    sku,
    warehouse_id,
    on_hand_qty,
    reserved_qty,
    available_qty,
    updated_at
FROM inventory
WHERE sku = 'SKU-123';
```

This helps confirm:

* current stock
* reserved stock
* available stock
* which warehouse owns the inventory
* when the record was last updated

---

## Step 3: Review Sync History

```sql
SELECT
    sku,
    source_system,
    target_system,
    sync_status,
    processed_at,
    error_message
FROM inventory_sync_log
WHERE sku = 'SKU-123'
ORDER BY processed_at DESC;
```

I would look for:

* failed sync jobs
* unusual gaps between successful updates
* repeated error messages
* changes around the time inventory reached zero

---

## Step 4: Compare Systems

If both values are available in reporting tables:

```sql
SELECT
    i.sku,
    i.available_qty AS internal_qty,
    m.available_qty AS marketplace_qty,
    m.updated_at AS marketplace_updated_at,
    i.updated_at AS internal_updated_at
FROM internal_inventory i
JOIN marketplace_inventory m
    ON i.sku = m.sku
WHERE i.sku = 'SKU-123';
```

The mismatch itself is useful, but the timestamps are just as important.

If the internal quantity updated recently while the marketplace record is several hours old, the issue is more likely related to synchronization than inventory calculation.

---

## Step 5: Possible Root Causes

At this point I would organize the likely causes.

### More likely

* Inventory synchronization failed
* Marketplace update was delayed
* Integration job stopped processing
* Wrong warehouse inventory was sent

### Possible

* SKU mapping changed
* Reserved inventory logic differs between systems
* Manual adjustment did not trigger a marketplace update
* Cached marketplace quantity was reused

### Needs evidence

I would not label any cause as confirmed until I had logs, timestamps, or system records supporting it.

---

## Step 6: How I Would Use AI

At this stage, I might provide the confirmed facts and available data to an AI tool and ask it to help organize the investigation.

Useful tasks:

* rank possible causes
* suggest additional checks
* identify missing information
* draft a short status update
* convert technical findings into business language

I would not ask AI to decide which system is correct.

That decision should come from the source systems, warehouse records, and business rules.

---

## Step 7: Immediate Action

If the warehouse confirms there is no inventory and the marketplace still shows stock, the priority is preventing additional overselling.

Possible immediate actions:

1. Correct the marketplace quantity
2. Temporarily stop sales for the affected SKU if needed
3. Identify orders accepted after inventory reached zero
4. Notify operations or customer support if cancellations may be required
5. Continue investigating the synchronization failure

The technical root cause is important, but the immediate business impact should be contained first.

---

## Step 8: Stakeholder Update

Example:

> We identified an inventory mismatch affecting SKU-123.
>
> Internal inventory and warehouse records both show zero available units, while the marketplace is still showing 120 units available.
>
> The issue appears to be related to inventory synchronization, but the exact cause is still being investigated. We are reviewing recent sync records and working to correct the marketplace quantity.
>
> Until the quantity is corrected, there is a risk of additional orders being accepted without available inventory.
>
> We are also identifying any orders placed after inventory reached zero so Operations can determine whether follow-up action is required.

---

## Step 9: Longer-Term Prevention

After resolving the immediate issue, I would look at ways to reduce the chance of recurrence.

Possible improvements:

* Alert when internal and marketplace inventory differ above a threshold
* Alert when inventory sync jobs fail repeatedly
* Monitor how long marketplace inventory has not updated
* Add reconciliation reporting across channels
* Review SKU and warehouse mapping regularly
* Create an exception dashboard for high-risk mismatches

Example monitoring logic:

```sql
SELECT
    i.sku,
    i.available_qty AS internal_qty,
    m.available_qty AS marketplace_qty,
    ABS(i.available_qty - m.available_qty) AS qty_difference
FROM internal_inventory i
JOIN marketplace_inventory m
    ON i.sku = m.sku
WHERE ABS(i.available_qty - m.available_qty) >= 5;
```

This would not automatically fix the problem, but it could help Operations detect mismatches before they result in a larger number of incorrect orders.

---

## What This Example Demonstrates

This case is less about writing a complicated SQL query and more about combining:

* business impact
* data validation
* system thinking
* root-cause analysis
* stakeholder communication
* AI-assisted investigation
* operational follow-up

The goal is to move from **“the numbers do not match”** to **“we know what is affected, what needs to be checked, what needs to be contained, and what should happen next.”**
