# Product Vision — Simple Stock Flow

> **Single source:** [`spec/data-model.md`](../spec/data-model.md). The vision is formulated **from what the model protects and what it leaves out**. Where the model does not back it, it is marked **[ASSUMPTION S-nn]** ([`../assumptions.md`](../assumptions.md)).
> Problem it solves: [`problem-framing.md`](problem-framing.md).

---

## 1. Vision statement

**For** the internal operators of a small shop [S-04]

**who** need to keep inventory and sales records without depending on each person following the rules by hand

**Simple Stock Flow**

**is a** stock control and sales registration system

**that** keeps stock correct, leaves an unalterable record of each sale and allows querying what was sold per product

**unlike** keeping those accounts by hand, where a rename, a price change or two simultaneous sales can alter the past or leave inventory negative

**our product** protects every business rule in more than one layer and honestly declares which ones still live only in code.

---

## 2. Pillars

| Pillar | What it means | Backing in the model |
|---|---|---|
| **Reliable stock** | Inventory is never negative and simultaneous sales do not overwrite each other | §2.2, D-04, ADR-002 |
| **Faithful history** | What was sold does not change: frozen values, immutable sales, stable reports | §1, §2.3, §2.4, §11.1 |
| **Minimal scope** | Only what the business asked for: five product attributes, five categories, one currency, no auditing or customers | DP-03, D-10, D-05, §8, §1 |
| **Verifiable truth** | The documentation can be checked against the engine; if it disagrees, the engine wins | §10, article X |
| **Sufficient security** | Two roles, protected credentials, small privacy surface | §2.5, §7, §9.2 |

---

## 3. Product principles

They guide decisions when a choice must be made. All come from decisions the model already closed.

1. **A rule that lives only in code is declared debt, not a guarantee.** What is not enforced in the engine is marked *pending*; it is not promised (§«How to read»).
2. **The past is not rewritten.** If something changes in the catalog, sales already made keep saying what they said (§1, §11.1).
3. **Scope is not invented.** No requirement, no column, table or attribute (§8, DP-03).
4. **Values are set by the domain, not the engine.** A second source of truth that nobody tests is worse than none (§3, §8).
5. **A declared lie is debt. A silent lie is a trap** (§13).

---

## 4. Status and evolution

The model defines no product horizons or dates. All it gives is **planned work** with a task id. It is kept as is, without inventing a calendar:

| Task | What it delivers | Status according to the model | Citation |
|---|---|---|---|
| T-02 | Initial schema, category seed, rename to singular | Done (four migrations applied) | §3.2 |
| T-05 | Explicit currency guard in `Sale.AddItem` | Pending | §2.3 |
| T-09 | Soft delete (`deleted_at` + global filter) | Done (settled 2026-09-20) | §2.2, §13 D-1 |
| T-10 | `xmin` as concurrency witness | Cited as done: the stock `CHECK` and `xmin` exist | §3, §3.2 |
| T-11 | Frozen `sale_item.category_name` | Cited as `engine` in §2.4; its column appears as pending in §3 | §2.4, §3 |
| T-12 | `sold_by_user_id` + `FK-4`; rename `sold_by` → `sold_by_username` | **Pending** (confirmed in §13) | §3, §5, §13 |
| T-13 | Three missing indexes and removal of `IX_sale_item_sale_id` | Pending | §6.2 |
| T-20 | Five `CHECK`s + `sale_id NOT NULL` + composite unique + `FK-3` | Partial: the last three settled; the `CHECK`s unconfirmed [S-10] | §4, §13 D-2 |

> **Note.** The status of T-10 and T-11 is hard to read because §2.4/§3 and §13 were written at different moments (see [`../05-architecture/verification.md`](../05-architecture/verification.md)).

---

## 5. Definition of achieved

> The product is achieved when stock is correct under real use, the sales history cannot be altered and reports of closed periods do not change.

| Key result | Measurement | Citation |
|---|---|---|
| **KR1.** Stock never goes negative, even with simultaneous sales | Concurrency test on the same product | D-04, ADR-002 |
| **KR2.** No sale can be modified or deleted | No edit or delete port exists | §2.3 |
| **KR3.** A report of a closed period does not change after renaming, repricing or recategorizing | Compare the report before and after | §11.1 |
| **KR4.** Every model rule is enforced in the layer it says | The three §10 queries match | §10 |
| **KR5.** All declared debts are settled or remain visible | §13 register | §13 |

> [ASSUMPTION S-17] The five key results are our reading of what would make the product «achieved». The model defines no KRs or goals.

---

## 6. Correlations

- Problem (the why) → [`problem-framing.md`](problem-framing.md)
- Stories that implement the vision → [`../04-requirements/user-stories.md`](../04-requirements/user-stories.md)
- Quality attributes → [`../04-requirements/non-functional.md`](../04-requirements/non-functional.md)
- Domain → [`../02-domain/entities-and-rules.md`](../02-domain/entities-and-rules.md)
