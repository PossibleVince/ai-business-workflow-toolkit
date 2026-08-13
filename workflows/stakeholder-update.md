# Stakeholder Update

A simple template for turning a technical or operational issue into a short update that business teams can actually use.

I use this type of structure when the investigation is still ongoing but stakeholders need a clear status update.

## What the update should answer

A good update should quickly answer:

* What happened?
* What is affected?
* How serious is the issue?
* What do we know so far?
* What has already been done?
* What still needs to be checked?
* What happens next?

The goal is to avoid sending raw logs, SQL output, or long technical explanations unless they are actually needed.

---

## Prompt Template

```text
I need to write a short stakeholder update for an operational issue.

Audience:
[Operations / Finance / Management / Engineering / Customer Support]

Issue:
[Describe the issue]

Business impact:
[Orders, customers, revenue, inventory, reporting, etc.]

Confirmed facts:
- [fact 1]
- [fact 2]
- [fact 3]

Actions already taken:
- [action 1]
- [action 2]

Still being investigated:
- [open question 1]
- [open question 2]

Next steps:
- [next step 1]
- [next step 2]

Please write a concise business-facing update.

Requirements:
- Keep technical details minimal unless they affect the decision.
- Clearly separate confirmed facts from assumptions.
- Do not imply the issue is resolved unless it is confirmed.
- Include the current impact, actions taken, and next steps.
- Keep the tone calm and factual.
```

---

## Example: Marketplace Inventory Sync Issue

### Internal Technical Notes

The internal inventory table shows quantity = 0 for SKU-123.

Amazon still shows 120 units available.

The warehouse confirmed there is no physical inventory.

The latest successful inventory sync was approximately four hours ago.

Several later sync attempts show failed status.

The cause of the failed sync is still under investigation.

### Poor Update

> There is an inventory API synchronization issue. The inventory table is correct but the Amazon quantity is stale because several sync jobs failed. We are checking the logs and API response.

This is technically useful, but it does not tell the business team enough about the actual impact or what they should expect next.

### Better Update

> We identified an inventory mismatch affecting SKU-123. Our internal inventory and warehouse records both show that the item is out of stock, while Amazon is currently showing 120 units available.
>
> The issue appears to be related to recent inventory synchronization failures, but the underlying cause is still being investigated.
>
> We are reviewing the failed sync records and working to correct the marketplace quantity. Until the quantity is updated, there is a risk that additional orders may be accepted for inventory that is not available.
>
> We will provide another update once the marketplace inventory has been corrected and the sync issue has been validated.

---

## Example: Shipping Cost Increase

### Situation

Average shipping cost increased from approximately $8.10 to $9.40 per order over the last two weeks.

The increase appears concentrated in oversized packages and West Coast-to-East Coast shipments.

No carrier pricing change has been confirmed yet.

### Example Update

> Average shipping cost has increased by approximately $1.30 per order over the last two weeks.
>
> Initial analysis shows that most of the increase is coming from oversized shipments and longer-distance delivery zones. We have not yet confirmed whether the change is related to carrier pricing, packaging decisions, or order mix.
>
> We are reviewing shipment dimensions, service levels, and carrier charges to isolate the main drivers. Once that review is complete, we will identify whether the issue requires a carrier discussion, packaging change, or operational adjustment.

---

## When I Would Not Use AI

I would not send an AI-generated update without reviewing it first.

Before sharing the message, I would verify:

* Numbers and dates
* Issue status
* Business impact
* Whether the problem is actually resolved
* Whether any statement is still only a hypothesis
* Whether sensitive internal information should be removed

AI is useful for organizing the message, but the final update should still reflect the actual system and business status.
