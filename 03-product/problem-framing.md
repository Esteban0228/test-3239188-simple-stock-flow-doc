# Problem Definition — Simple Stock Flow

> **Single source:** [`spec/data-model.md`](../spec/data-model.md). A data model does not describe a market: it describes what a system **protects**. The problem is reconstructed from **what the model takes the trouble to protect**. Where the model gives no evidence, the document says so.
> **Assumption** = something that does not come from the model: **[ASSUMPTION S-nn]**, explained in [`../assumptions.md`](../assumptions.md).

---

## 1. The problem in one sentence

A **small shop** that sells items from a catalog, served by **internal operators**, needs to know **how much stock is left** and **what it has sold**, without the history changing when it renames, reprices or recategorizes a product, and without two simultaneous sales leaving inventory negative. Today that information depends on each person following the rules by hand.

> [ASSUMPTION S-04] That the shop is a hardware store comes only from the names of the five seeded categories (*General, Herramientas, Electricidad, Fontanería, Pinturas*, §9.1). The model **does not name** the business or its sector. The problem definition does not depend on that assumption.

---

## 2. Which places of the model the problem comes from

Each statement of the problem has **backing in a model rule**. If the problem did not exist, the rule would make no sense.

| Part of the problem | What the model protects | Citation |
|---|---|---|
| «How much stock is left» must be true | `stock >= 0` after any operation; withdrawing more than available **fails**; the `CHECK` is the last barrier | §2.2, §4, ADR-002 |
| Two simultaneous sales must not fight over the same product | Optimistic concurrency with `xmin`; Q3 is the contention point | §3, §6.1, D-04 |
| «What was sold» must not be alterable | The sale is an «accomplished and immutable commercial fact»; no edit or delete port exists | §1, §2.3 |
| Renaming or repricing must not rewrite the past | Price, product name and category name are **frozen** in the sale line | §1, §2.4 |
| A withdrawn product must not break old reports | Soft delete; `FK-3` prevents physical deletion | §2.2, §5 |
| The report must be stable | A closed report never changes; it is grouped by the frozen label | §11.1 |
| We need to know **who** registered each sale | `sold_by` required; authorship without orphans (`FK-4`, pending T-12) | §2.3, §5 |
| Not everyone can do everything | Two roles, `admin` and `seller`; the `admin` role is not granted at runtime | §2.5, DP-04 |

---

## 3. Affected users

| Segment | Description | Estimated size | Priority |
|---|---|---|---|
| **Seller** (`seller`) | Operator who registers sales and queries the catalog | Not defined in the model | High |
| **Administrator** (`admin`) | Operator who maintains the catalog and creates sellers [S-03] | Not defined in the model | High |

**The buyer is not a user of the system.** There is no customer or buyer entity and no end-customer personal data is stored (§1, §7).

### Job each one tries to do (JTBD)

**Seller.** When I serve someone at the counter, I want to register what they take and have inventory go down by itself, so I do not sell what I no longer have nor have to correct afterwards.

**Administrator.** When a price, a name or a category changes, or I withdraw a product, I want past sales to keep saying what they said, so I can trust reports of closed periods.

> [ASSUMPTION S-15] Both phrases are **reconstructions**: the model includes no interviews or usage descriptions. They are formulated from the rules of §2.2–§2.4 and §11.1.

---

## 4. Evidence of the problem

| Type | Source | Finding |
|---|---|---|
| Model rules | `spec/data-model.md` §2–§5, §11.1 | Each rule protects against a concrete failure (negative stock, altered sale, changing report). It is **indirect** evidence of what is feared to happen |
| Measured defects | §13, §11.1 | The document itself records lies and defects measured against the engine (for example A-1, anonymous user registration, and A-7, product-name tie-break). Evidence that **these failures already happened or were one step away** |
| Interviews, surveys, observation of real work | **Do not exist in the delivered material** | No direct evidence from users |
| Usage figures | **Do not exist** | Current volume is 5 categories, 1 user and 0 products, sales and lines (§10.4) |

**Note.** No user research is available. The problem rests on what the system protects, not on what a user said. Any extension should be validated with whoever operates the shop before being treated as fact.

---

## 5. User's current solution

**The model does not describe it.** It does not say how inventory or sales records are kept today. The usual alternatives of a small shop —notebook, spreadsheet— are **not backed** by the delivered material and are not listed as evidence [ASSUMPTION S-16].

---

## 6. Solution hypothesis

**We believe that** an internal system with a product catalog, an immutable sales record and a sales-by-product report, protected by two roles,

**for** the operators of a small shop,

**will achieve** that inventory reflects real sales and that the commercial history can be queried without changing along with the catalog.

**We will know we were right when** checks that the model itself makes verifiable are met:

| Check | Citation |
|---|---|
| Two simultaneous sales of the last unit: only one succeeds and stock never goes negative | D-04, ADR-002 |
| The report of a closed period gives the same result before and after renaming or recategorizing a product | §11.1 |
| A registered sale cannot be modified or deleted by any application path | §2.3 |
| The §10 queries match what the document states | §10 |

> [ASSUMPTION S-17] Turning those checks into product success metrics is our reading. The model defines no product metrics.

---

## 7. What the product does not intend to solve

Everything below is **decided and not reopened** (§12):

| Not solved | Decision | Citation |
|---|---|---|
| Customer or buyer management | The entity does not exist | §1 |
| Multi-currency sales | Single-currency | D-05 |
| Additional product attributes | Closed | DP-03 |
| Report by seller | Decided: no | DP-02 |
| Catalog change auditing | Closed, no requirement | §8 |
| Payments, cards, health data | Do not apply | §7 |
| Category maintenance | No CRUD | D-10 |
| Who grants the `admin` role at runtime | Nobody | DP-04 |

---

## 8. Risks and open questions

| # | Question | Status | Citation |
|---|---|---|---|
| H-2 | What policy exists for orphaned image binaries? | **Open**: operational, no impact on the deliverable | §11 |
| — | How many users, products and sales are expected? | **The model does not say** | §10.4 |
| — | Which role can manage the catalog and query reports? | **Not defined** [S-03] | — |
| CA-06.1 | `spec.md` CA-06.1 says «one row per product»; H-1 produces more than one if there was a recategorization | **Owner decision pending** | §11.1 |
