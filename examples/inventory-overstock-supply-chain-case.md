# Inventory Overstock & Supply Chain Recovery Case Study

This case is based on a real operational problem I worked around during a period of unusually high logistics costs, changing demand, and limited warehouse capacity.

Some company-specific details, system names, and exact financial values have been simplified or omitted.

## Business Problem

The company had committed to inventory purchases based on earlier demand expectations.

As market conditions changed, several problems happened at the same time:

* Product demand slowed
* International shipping costs increased
* Tariff and import-related costs increased
* Existing inventory moved more slowly than expected
* Warehouse capacity became constrained
* Additional containers continued arriving from overseas suppliers

The result was a capacity problem.

Inventory already in the warehouse was not selling quickly enough to create space for incoming products.

At the same time, containers arriving at the port could not always be moved into the warehouse immediately.

Leaving containers or freight at the port created additional storage and detention-related costs.

The problem was no longer simply:

> “We have too much inventory.”

It had become a broader cash-flow and supply-chain issue.

---

## Operational Impact

The situation affected several parts of the business:

### Warehouse

Available storage space was limited.

Incoming inventory could not be received at the normal pace.

### Logistics

Containers waiting at the port created additional storage-related costs.

Transportation scheduling became harder because there was nowhere to put additional inventory.

### Purchasing

Continuing normal supplier shipments would make the capacity problem worse.

### Sales

Slow-moving inventory needed to be sold faster, even if that meant accepting lower margins on some products.

### Cash Flow

Cash was tied up in inventory while the company continued paying logistics, storage, and import-related costs.

---

## My Role

I supported the decision-making process from the data and operations side.

The work included:

* Reviewing inventory levels and product movement
* Identifying slow-moving and high-risk inventory
* Comparing sales velocity with incoming inventory
* Supporting warehouse-capacity discussions
* Analyzing logistics and storage cost pressure
* Helping prioritize products for clearance
* Providing reporting that helped management decide where to slow or pause replenishment

The final purchasing, pricing, and warehouse decisions were made by management and the relevant business teams.

My role was to make the situation easier to measure and prioritize.

---

## Step 1: Understand Inventory Exposure

The first step was separating inventory by how quickly it was actually moving.

A simplified analysis might include:

```sql
SELECT
    sku,
    category,
    on_hand_qty,
    avg_weekly_sales,
    CASE
        WHEN avg_weekly_sales > 0
        THEN on_hand_qty / avg_weekly_sales
        ELSE NULL
    END AS weeks_of_supply
FROM inventory_summary;
```

Products with high inventory and low sales velocity were the biggest concern.

I would pay particular attention to:

* High-value inventory
* Large or bulky products
* Products with very low sales velocity
* Seasonal products
* Items already taking significant warehouse space
* Products with more inventory still in transit

---

## Step 2: Identify Slow-Moving Inventory

A simple prioritization could look at:

* Units on hand
* Days since last sale
* Average weekly sales
* Gross margin
* Storage footprint
* Additional units already in transit

For example:

```sql
SELECT
    sku,
    on_hand_qty,
    days_since_last_sale,
    avg_weekly_sales,
    inventory_value
FROM inventory_risk
WHERE on_hand_qty > 0
ORDER BY
    avg_weekly_sales ASC,
    inventory_value DESC;
```

The goal was not just to find products with the largest quantity.

A product with fewer units but very low demand and high storage requirements could be a larger operational problem.

---

## Step 3: Connect Inventory With Incoming Supply

Existing inventory could not be analyzed by itself.

The company also needed visibility into what was already:

* Ordered from suppliers
* In production
* In transit
* At the port
* Waiting for warehouse receiving

A simplified view:

| SKU       | On Hand | Avg Weekly Sales | In Transit | Weeks of Supply |
| --------- | ------: | ---------------: | ---------: | --------------: |
| Product A |   1,200 |               35 |        600 |            34.3 |
| Product B |     350 |              120 |        100 |             2.9 |
| Product C |     900 |               10 |        500 |            90.0 |

Product C would clearly require more attention than Product B, even though both had incoming inventory.

---

## Step 4: Estimate the Cost of Doing Nothing

One of the important business questions was:

> Is it cheaper to protect margin and hold the inventory, or sell it faster and free warehouse capacity?

Holding inventory had costs beyond the original purchase price.

These could include:

* Warehouse storage
* Port storage
* Container detention / demurrage exposure
* Additional handling
* Financing / cash tied up in inventory
* Product obsolescence
* Future markdown risk

This changed the pricing conversation.

Selling an item at a lower margin could sometimes be economically better than continuing to pay storage and logistics costs for inventory that was not moving.

---

## Step 5: Support a Short-Term Containment Plan

The company ultimately used several actions rather than relying on one solution.

### Slow or Pause Supplier Shipments

For products with excessive inventory, additional supplier shipments were delayed or paused where possible.

This prevented the existing capacity problem from becoming larger.

### Accelerate Clearance

Slow-moving products were prioritized for:

* Discounts
* Promotions
* Marketplace pricing adjustments
* Clearance strategies

The goal was to convert inventory back into cash and warehouse space.

### Add Temporary Storage Capacity

Temporary warehouse space provided short-term capacity while the existing inventory was being reduced.

This was not the ideal long-term solution, but it was preferable to allowing freight to remain at the port while storage charges continued accumulating.

### Prioritize Incoming Inventory

Not all incoming products had the same business value.

Higher-demand products could be prioritized for warehouse space while lower-priority inventory was delayed or redirected when possible.

---

## Step 6: Build a Simple Inventory Risk View

A useful dashboard or recurring report could rank inventory based on several factors.

Example:

```text
Inventory Risk Score =
High Weeks of Supply
+ Low Sales Velocity
+ High Inventory Value
+ High Storage Footprint
+ Incoming Units
```

The exact score does not need to be mathematically complex.

Its main purpose is to help Operations and Management focus attention on the products creating the largest combined financial and capacity risk.

---

## Step 7: Track Recovery

After actions were taken, I would monitor:

* Total inventory units
* Inventory value
* Weeks of supply
* Warehouse utilization
* Units in transit
* Slow-moving inventory
* Clearance sales
* Average shipping / storage cost
* Products requiring additional purchase holds

The important part was seeing whether warehouse pressure was actually improving, rather than assuming that discounts alone solved the problem.

---

## Where AI Could Help Today

If I were handling a similar situation today, AI could help accelerate parts of the analysis.

For example:

* Summarize which SKUs have the highest combined inventory risk
* Draft management summaries from operational metrics
* Suggest additional questions for procurement and logistics teams
* Generate SQL starting points for inventory segmentation
* Compare different clearance scenarios
* Draft supplier or internal communication
* Turn weekly metrics into a concise risk report

I would still rely on actual inventory, purchasing, warehouse, and finance data for the final decisions.

AI can help organize the problem. It should not independently decide whether a shipment should be cancelled or how aggressively a product should be discounted.

---

## Business Outcome

The response focused on containing the problem rather than waiting for demand to recover.

The company:

* Slowed or paused selected supplier shipments
* Accelerated the sale of excess inventory
* Used temporary warehouse capacity to reduce immediate port pressure
* Prioritized warehouse space more carefully
* Increased visibility into slow-moving inventory and incoming supply

These actions helped reduce the risk of continuing storage charges and prevented additional inventory from worsening the warehouse-capacity problem.

---

## What This Case Demonstrates

This was not purely a data-analysis problem.

It required understanding how several business functions were connected:

**Demand → Purchasing → International Logistics → Warehouse Capacity → Pricing → Cash Flow**

The useful part of the analysis was not producing another inventory report.

It was helping the business answer:

* Which inventory is actually creating the most risk?
* Which incoming shipments make the problem worse?
* Where should purchasing slow down?
* When is a lower-margin sale better than holding inventory?
* How can the company create warehouse capacity quickly?
* Which operational costs should be contained first?

That combination of data analysis and business context is what made the work useful.
