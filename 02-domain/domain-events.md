# Domain Events — Simple Stock Flow

> **Single source:** [`spec/data-model.md`](../spec/data-model.md). **The model defines no domain events**: it does not mention them, nor name brokers, queues or consumers. This document says so plainly and proposes, **as an assumption**, which business facts *could* be declared as events if it were ever needed.
> **Assumption** = something that does not come from the model: **[ASSUMPTION S-06]**, in [`../assumptions.md`](../assumptions.md).

---

## 1. What the model says about events

| Question | Answer | Citation |
|---|---|---|
| Does it define domain events? | **No** | whole document |
| Are there event consumers? | **None named**: there is no external analytics or export | §7.1 |
| Is there messaging between processes? | **No evidence**: one schema and one API repository | §3, §12, S-02 |
| Is there a historical change record? | **Closed decision: no.** No `created_at`/`updated_at` or change log | §8 |
| Where does what the business needs to know about the past live? | In two instants: `sale.sold_at` and `product.deleted_at` | §8 |

**Consequence.** The system **needs no event infrastructure**. The facts that matter to the business are already stored as **immutable state**: a completed sale is itself the record of the fact (§1, §2.3).

---

## 2. Principles if they are declared

A domain event would be a **business fact that already happened**, in past tense, with minimal payload. For this system, also:

- **No personal data.** `username` and `sold_by` are personal data (§7), so the payload carries **identifiers**, not names.
- **No password hash**, ever (§7).
- **No edited sales.** Sales are neither edited nor deleted: there is **no** sale-modified or sale-voided event (§2.3).
- **No assumed infrastructure.** This document defines no brokers, queues, delivery guarantees or consumers (S-02).

---

## 3. Candidate facts [ASSUMPTION S-06]

Derived from the operations the model names for each aggregate. **None is published today.**

| Fact | Aggregate | Source operation | Minimal payload | Why it might matter | Citation |
|---|---|---|---|---|---|
| `SaleRegistered` | `Sale` | `Sale.AddItem` + confirmation | `saleId`, `soldAt`, `itemCount` | It is the accomplished, immutable commercial fact | §1, §2.3 |
| `StockWithdrawn` | `Product` | `Product.Withdraw` | `productId`, `quantity` | Stock change | §2.2 |
| `StockReplenished` | `Product` | `Product.Restock` | `productId`, `quantity` | Stock change | §2.2 |
| `ProductPriceChanged` | `Product` | `Product.ChangePrice` | `productId` | It is the fact that requires earlier lines to keep their frozen price | §2.2, §1 |
| `ProductDeactivated` | `Product` | soft delete | `productId`, `deactivatedAt` | Triggers deletion of the image binary | §2.2, §7.1 |
| `UserRegistered` | `User` | user registration | `userId`, `role` | Seller registration | §2.5, DP-04 |

**Note on `ProductDeactivated`.** The model requires an order when deleting the binary: first `image_key` is set to null and the transaction is committed; then the binary is deleted (§7.1). An event does not change that order nor provide atomicity between the database and external storage, which **the model declares unattainable** (§7.1).

---

## 4. Facts that do NOT exist

| No event for… | Because | Citation |
|---|---|---|
| Sale modified or voided | The sale is immutable | §2.3, §7.1 |
| Category created, renamed or deleted | There is no category maintenance | §2.1, D-10 |
| `admin` role granted at runtime | Nobody grants it | DP-04 |
| Report generated | The report is not persisted; it is computed | §1, D-06 |
| Password changed | The model defines no such operation [S-06] | — |

---

## 5. Decision

**No event infrastructure is adopted.** If a real consumer appears later, the six facts of §3 are the starting point; **before that**, the messaging decision would have to be opened in `05-architecture`.
