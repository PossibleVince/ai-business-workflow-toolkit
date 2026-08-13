# UAT Checklist

A practical template for turning business requirements into User Acceptance Testing scenarios.

I use this structure to make sure a feature works not only in the expected case, but also around validation rules, permissions, edge cases, and existing workflows.

## Before Testing

Confirm:

* What business problem the feature is solving
* Who is allowed to use the feature
* Required vs. optional fields
* Validation rules
* Expected system behavior
* What data should be created or updated
* What should happen when something fails
* Existing workflows that could be affected

---

## Prompt Template

```text
I need to create a UAT checklist for a business system change.

Feature:
[Describe the feature]

Business goal:
[Why the feature is needed]

Users / roles:
[List the users who can access it]

Requirements:
- [requirement 1]
- [requirement 2]
- [requirement 3]

Business rules:
- [rule 1]
- [rule 2]

Data affected:
[List important fields, tables, reports, or systems]

Please create UAT scenarios covering:

1. Happy path
2. Required-field validation
3. Invalid inputs
4. Boundary conditions
5. User permissions
6. Data consistency
7. Error handling
8. Existing workflow regression
9. Reporting / downstream impact if applicable

For each scenario include:
- Test scenario
- Preconditions
- Steps
- Expected result
- Priority

Do not invent business rules that are not provided.
List any missing requirement that needs clarification before testing.
```

---

## Example: Order Cancellation Workflow

### Requirement

Operations needs the ability to cancel an order before it is released to the warehouse.

Business rules:

* Only Operations users can cancel orders
* Shipped orders cannot be cancelled
* A cancellation reason is required
* Inventory should be returned to available stock
* Cancelled orders should no longer appear in the warehouse fulfillment queue

---

## Example UAT Scenarios

| Scenario                    | Preconditions                                  | Expected Result                                          | Priority |
| --------------------------- | ---------------------------------------------- | -------------------------------------------------------- | -------- |
| Cancel eligible order       | Order is not shipped; user has Operations role | Order status changes to Cancelled                        | High     |
| Missing cancellation reason | Eligible order; reason left blank              | System blocks cancellation and shows validation message  | High     |
| Cancel shipped order        | Order status = Shipped                         | Cancellation is rejected                                 | High     |
| Unauthorized user           | User does not have Operations permission       | Cancel action is unavailable or rejected                 | High     |
| Inventory restoration       | Order reserves 3 units                         | 3 units return to available inventory after cancellation | High     |
| Fulfillment queue update    | Order is currently in warehouse queue          | Cancelled order is removed from fulfillment queue        | High     |
| Duplicate cancellation      | Order already cancelled                        | System does not process cancellation twice               | Medium   |
| Reporting consistency       | Cancelled order exists                         | Reporting reflects cancelled status correctly            | Medium   |

---

## Additional Checks

### Data

* Order status is updated correctly
* Cancellation reason is stored
* Inventory quantity is restored once
* No duplicate transactions are created
* Updated timestamps are correct

### Permissions

* Operations user can perform the action
* Unauthorized users cannot perform the action
* Direct system/API access should not bypass the same rule

### Error Handling

Test what happens if:

* Inventory update fails
* The order status changes while the user has the page open
* The system times out during cancellation
* A downstream service is temporarily unavailable

The user should not receive a false success message when part of the operation failed.

---

## Regression Checks

After the new workflow passes its direct tests, I would also check:

* Normal order creation still works
* Warehouse fulfillment still receives eligible orders
* Inventory reservation still works for active orders
* Existing reports still calculate order status correctly
* Other order-management actions are unaffected

---

## Using AI for UAT

AI is useful for finding test scenarios that may be easy to miss, especially around:

* Boundary conditions
* Permissions
* State transitions
* Failure scenarios
* Regression risks

However, AI does not know the actual business rules unless they are provided.

Before using generated test cases, I would compare them against:

* Business requirements
* Existing system behavior
* Data contracts
* Stakeholder expectations

Any unclear requirement should become a question for the business team rather than an assumption in the test plan.
