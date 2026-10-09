# General Description — Simple Stock Flow

> **Single source:** [`spec/data-model.md`](../spec/data-model.md). It is the **only** input of the challenge: the other documents of the original project (`constitution.md`, `spec.md`, `plan.md`, `adr/`, `tasks.md`, `api-contract.md`) are not delivered (challenge README). Whatever does not come from the model is marked **[ASSUMPTION S-nn]** ([`../assumptions.md`](../assumptions.md)).
> **Language:** these documents are in English; code and column identifiers are kept in English, as the glossary requires [ASSUMPTION S-01]. The model and the challenge README are in Spanish, so quotes («…») are translations.

---

## 1. What is Simple Stock Flow?

An internal system to **keep the catalog, stock and sales** of a small shop. It is operated by the business's own staff —**not its customers**— and relies on a PostgreSQL database.

What the model makes clear that it does:

- Maintains a **product catalog** with name, price, stock, category and optional image (§1, DP-03).
- **Registers sales** of several lines that withdraw stock and become **immutable** (§1, §2.3).
- Produces a **sales report by product** over a date range (§1, §6.1 Q9).
- Distinguishes **two roles** of internal user: `admin` and `seller` (§2.5).

> [ASSUMPTION S-04] That it is for a hardware store comes only from the five seeded categories (*General, Herramientas, Electricidad, Fontanería, Pinturas*, §9.1). The model **does not name** the business. The category names are seed data and are kept in their original Spanish.

---

## 2. Problem it solves

### Before the system

When inventory and sales depend on each person following the rules by hand:

- Two simultaneous sales can leave stock **negative**.
- Renaming, repricing or recategorizing a product **rewrites the past** of the reports.
- A sale can be **edited or deleted** after being made.
- It is unclear **who** registered each operation.

### With the system

- Stock **never drops below zero** and simultaneous sales do not overwrite each other (§2.2, D-04).
- Each sale line **freezes** the price, name and category of the moment (§2.4).
- Sales are **accomplished facts**: they are neither edited nor deleted (§2.3).
- Each sale is attributed to a user (§2.3).

The problem is detailed in [`../03-product/problem-framing.md`](../03-product/problem-framing.md).

---

## 3. Users

| Role | Description | What it does in the system |
|---|---|---|
| **Seller** (`seller`) | Internal operator who serves sales | Authenticates, queries the catalog and **registers sales** (§1, §2.5) |
| **Administrator** (`admin`) | Internal operator with more privilege | Creates sellers (DP-04); maintains the catalog [ASSUMPTION S-03] |
| **Deployment owner** | Whoever configures the environment | Provides the credentials with which startup creates the first administrator (§9.2) |

**There are no customers in the system.** The entity does not exist and no buyer personal data is stored (§1, §7). The operator's `username` is personal data (§7).

---

## 4. Main flow

```text
Administrator                              Seller
     |                                        |
     v                                        v
Maintains the catalog                    Authenticates
(products, price, stock,                      |
 one of five categories, image)               v
     |                                  Searches products
     |                                        |
     |                                        v
     |                              Registers a sale (≥ 1 line)
     |                                        |
     |                     +------------------+------------------+
     |                     |                                     |
     |                     v                                     v
     |            Stock drops per line                Line freezes name,
     |            (a single operation)                price and category
     |                     |
     |                     v
     +------------> Sales report by product
                    (date range, computed in the engine)
```

Backing for each part of this flow: catalog §2.2; search §6.1 Q1; sale §2.3–§2.4; report §6.1 Q9 and §11.1.

---

## 5. The system in numbers (according to the model)

| Datum | Value | Citation |
|---|---|---|
| Entities / tables | 5: `category`, `product`, `sale`, `sale_item`, `user` | §2 |
| Aggregate roots | 3: `Product`, `Sale`, `User` | §2 |
| Schema | `sales`, in PostgreSQL 16 | «Verified against», §3 |
| Roles | 2: `admin`, `seller` | §2.5 |
| Categories | 5, seeded, no maintenance | §9.1, D-10 |
| Applied migrations | 4 | §3.2 |
| Volume at 2026-09-19 | 5 categories · 1 user · 0 products · 0 sales · 0 lines | §10.4 |

> **Note.** The model states «22 columns» in §3 and §12, but the measurement of §10.1 (2026-09-19) returns **21**. The probable difference is `product.deleted_at`, created after that measurement (T-09, §13 D-1) — this is a reading, not a fact of the model. See OBS-01 in [`../05-architecture/verification.md`](../05-architecture/verification.md).

---

## 6. Project status

The model describes a system **already built and under review**, with explicit pending tasks:

- **Pending:** sale authorship with a foreign key to `user` (T-12), three indexes (T-13), explicit currency guard (T-05) and pushing five invariants down to the engine (T-20, partial) (§2.3, §4, §6.2).
- **Open gap:** cleanup policy for orphaned image binaries (H-2) (§11).
- **Owner decision pending:** rewrite `spec.md` CA-06.1 so it does not contradict H-1 (§11.1).

Detailed in [`../03-product/vision.md`](../03-product/vision.md) and [`../05-architecture/verification.md`](../05-architecture/verification.md).

---

## 7. How to read this documentation

| Order | Folder | What it answers |
|---|---|---|
| 1 | [`01-context/`](.) | What the system is and is not |
| 2 | [`02-domain/`](../02-domain/) | Entities, rules, events and glossary |
| 3 | [`03-product/`](../03-product/) | Problem and vision |
| 4 | [`04-requirements/`](../04-requirements/) | User stories and non-functional requirements |
| 5 | [`05-architecture/`](../05-architecture/) | Style, components and where each rule lives; closing verification |
| — | [`../assumptions.md`](../assumptions.md) | Everything that does **not** come from the model |

> These documents were **reconstructed backwards**, from architecture to context, starting from the model. That is why the reading order is the reverse of the writing order.
