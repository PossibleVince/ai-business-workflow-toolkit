# Root Cause Analysis

This is a simple template I use to organize operational issues before jumping into fixes.

The goal is to separate what we already know from what still needs to be checked.

## When I would use this

* Order data does not match between systems
* Inventory numbers look wrong
* Shipping cost suddenly increases
* A report stops updating
* Orders stay in an unusual status for too long
* A business team reports a recurring issue but the cause is not clear

## Input

Before using AI, I try to provide:

* What happened
* When it started
* Which system or workflow is affected
* How many records / orders are impacted
* What changed recently
* What has already been checked
* Any sample records or error messages

## Prompt Template

```text
I am investigating an operational issue.

Issue:
[Describe the issue]

Business impact:
[What is affected and how serious it is]

Known facts:
- [fact 1]
- [fact 2]
- [fact 3]

What has already been checked:
- [check 1]
- [check 2]

Available data:
[List tables, reports, logs, or fields available]

Please help me organize the investigation.

Return:
1. Confirmed facts
2. Possible root causes, ranked by likelihood
3. What evidence would confirm or reject each cause
4. Data or system checks I should run next
5. Questions I should ask the business or operations team
6. Immediate workaround if one is appropriate
7. Longer-term fix ideas

Do not assume missing information is true.
Clearly label assumptions and unknowns.
```

## Example: Inventory Mismatch

### Situation

A marketplace shows 120 units available, but the internal inventory report shows 0.

Operations reports that several orders were accepted even though the warehouse says the item is out of stock.

### Known facts

* Marketplace quantity: 120
* Internal inventory quantity: 0
* Warehouse confirmed no physical stock
* Issue affects one SKU
* No recent manual inventory adjustment was reported

### What I would check first

1. Last successful inventory sync time
2. Whether the marketplace feed failed or was delayed
3. Whether the SKU mapping is correct
4. Whether inventory is being pulled from the wrong warehouse
5. Whether a cached value was reused
6. Whether inventory updates failed after a system change

### Example validation queries

```sql
SELECT
    sku,
    warehouse_id,
    available_qty,
    updated_at
FROM inventory
WHERE sku = 'SKU-123';
```

```sql
SELECT
    sku,
    sync_status,
    source_system,
    target_system,
    processed_at,
    error_message
FROM inventory_sync_log
WHERE sku = 'SKU-123'
ORDER BY processed_at DESC;
```

## How I would use AI here

I would use AI mainly to:

* organize possible causes
* help identify missing checks
* turn technical findings into a short update for operations
* draft follow-up questions

I would not use AI to decide which inventory number is correct without checking the source systems.

## Example Final Update

The current mismatch appears to be caused by a delayed or failed inventory synchronization between the internal inventory system and the marketplace.

The warehouse has confirmed that physical inventory is currently zero, while the marketplace is still showing 120 units available.

Next steps are to verify the most recent inventory sync, review any failed sync records, and temporarily prevent additional orders for this SKU until the marketplace quantity is corrected.

## Notes

This template is intentionally simple. In practice, the investigation may involve SQL, Excel, application logs, warehouse records, or communication with operations depending on the issue.
