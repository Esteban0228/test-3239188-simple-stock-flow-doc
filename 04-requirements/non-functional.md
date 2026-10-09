# Non-Functional Requirements — Simple Stock Flow

> **Single source:** [`spec/data-model.md`](../spec/data-model.md). A non-functional requirement appears here **only if the model demands or implies it**. Where the model is silent, the document says so and **does not invent figures** (see §3).
> NFRs express **what quality the system must meet**, not how it is implemented. The how is in [`../05-architecture/overview.md`](../05-architecture/overview.md).
> **Assumption** = something that does not come from the model: **[ASSUMPTION S-nn]**, explained in [`../assumptions.md`](../assumptions.md).

---

## 1. Summary

| ID | Quality attribute | Priority | Main citation |
|---|---|---|---|
| NFR-01 | Layered data integrity | High | «How to read», §4 |
| NFR-02 | Stock consistency under concurrency | High | §3 `xmin`, D-04, ADR-002 |
| NFR-03 | History immutability | High | §2.3, §7.1 |
| NFR-04 | Report stability | High | §1, §11.1 |
| NFR-05 | Monetary accuracy | High | §2.2, D-05 |
| NFR-06 | Credential protection | High | §2.5, §7, §9.2 |
| NFR-07 | Access control | High | §9.2, §13 D-3, DP-04 |
| NFR-08 | Personal data privacy | Medium | §7, DP-02 |
| NFR-09 | Retention | Medium | §7.1 |
| NFR-10 | Schema ownership and verifiability | High | §0, §3.2, §10, article X |
| NFR-11 | Performance of the most frequent queries | Medium | §6 |
| NFR-12 | Temporal consistency | Medium | §3 «Two cross-cutting rules» |
| NFR-13 | Maintainability and common language | Medium | §0, §1, article XI |

> [ASSUMPTION S-05] The **High / Medium** priority is our reading of the weight the model gives each attribute (how much text, how many barriers, how many closed decisions it devotes to it). The model assigns no priorities.

---

## 2. Requirements

### NFR-01 — Layered data integrity

**What is required.** Every invariant that can be expressed in the engine must be able to be pushed down to the engine; what today lives only in the domain is **declared pending**, not promised. An invariant that lives only in C# protects the application, not the data.

| Verifiable criterion | Citation |
|---|---|
| Each model rule carries exactly one of three marks: **engine**, **domain-only** or **pending (T-xx)** | «How to read» |
| If a rule marked **engine** does not appear in the engine, someone deleted it; if one marked **domain-only** appears, it must be reclassified | §10 |
| Declared as debt with a task: `price > 0`, `quantity > 0`, non-empty category name, valid `role` and lowercase username (T-20) | §4 |
| The engine is the **last barrier**: if a constraint trips, something wrote outside the adapter | «How to read», ADR-002 |

**Status of T-20 debt.** The model considers **settled** (2026-09-20) the three pieces of T-20 it cites in §13: `sale_item.sale_id NOT NULL`, the unique `(sale_id, product_id)` and `FK-3`. **It does not say the same about the five `CHECK`s** of §4, which remain marked «domain-only · T-20» [ASSUMPTION S-10: they are assumed **not pushed down** to the engine until measured].

---

### NFR-02 — Stock consistency under concurrency

**What is required.** Two simultaneous operations on the same product cannot leave stock negative or lose an update.

| Verifiable criterion | Citation |
|---|---|
| `product.stock >= 0` holds after any operation, also under concurrency | §2.2 |
| A barrier exists in the engine: `ck_product_stock_non_negative` | §4 |
| Conflict detection relies on `xmin`, which Postgres increments on every `UPDATE`, exposed as a shadow property | §3, D-04, T-10 |
| Batch product read (Q3) is the **contention point** that precedes the stock write | §6.1 |

---

### NFR-03 — History immutability

**What is required.** A registered sale is neither edited nor deleted, and its lines live and die with it.

| Verifiable criterion | Citation |
|---|---|
| No operation to edit or delete sales exists | §2.3, §7.1 |
| `FK-2` (`CASCADE`) exists so the model tells the truth about composition, even though «in practice it never fires» | §5 |
| A product with sale lines is not physically deleted: soft delete, and `FK-3` (`RESTRICT`) makes a manual delete fail loudly | §2.2, §5 |
| Sales and their lines are retained **indefinitely** | §7.1 |

---

### NFR-04 — Report stability

**What is required.** A report of a closed period **never changes**, even if a product is later renamed, repriced or recategorized.

| Verifiable criterion | Citation |
|---|---|
| The sale line stores a copy of the name, price and category name **at the moment of the sale** | §1 «Frozen name», §2.4 |
| `sale_item.category_name` has **no** foreign key, on purpose: renaming the category must not rewrite history | §2.4, §3 |
| The report groups by `product_id, product_name, category_name`; it does not pick a «winner» among labels | §11.1 |
| The report is not persisted: it is computed in the engine | §1, D-06 |

---

### NFR-05 — Monetary accuracy

**What is required.** Amounts are exact and consistent between domain and database; the system is single-currency.

| Verifiable criterion | Citation |
|---|---|
| An amount is rounded to **2 decimals** with `MidpointRounding.AwayFromZero` before saving | §2.2 |
| The column is `numeric(18,2)`; **if one changes, the other changes in the same migration** | §2.2, §3 |
| No table has a currency column | §3, D-05 |
| Total and subtotal are **computed**, not stored | §1, article VII |

---

### NFR-06 — Credential protection

**What is required.** The password never exists in plaintext inside the domain nor beyond the instant of verification, and no secret lives in the repository.

| Verifiable criterion | Citation |
|---|---|
| The domain never sees the plaintext password; the hash is produced by a port | §1 «Password hash», §2.5, D-09 |
| `password_hash` **never** appears in logs, responses, projections or error messages | §7 |
| `password_hash` is **never indexed**, and not for performance reasons | §6.3, §7 |
| No history or versioning of hashes is kept | §7.1 |
| The initial administrator is not seeded from SQL nor with a versioned literal hash | §9.2, article IX |

---

### NFR-07 — Access control

**What is required.** Only authenticated users operate the system; nobody obtains the `admin` role at runtime.

| Verifiable criterion | Citation |
|---|---|
| Without a bearer credential, protected operations answer 401 | §13 D-3 |
| With the `seller` role, user registration answers 403 | §13 D-3 |
| User registration is **not anonymous** (defect A-1, closed) | §9.2, §13 D-3 |
| The `admin` role is provisioned by the deployment from the environment; nobody grants it at runtime (DP-04) | §11 H-3 |
| Authorization policy lives in the **API**, not in the database nor the domain | §9.2 |
| `user.role` admits only `admin` and `seller` | §2.5 |

> [ASSUMPTION S-08] The model names neither the authentication mechanism nor the session duration.

---

### NFR-08 — Personal data privacy

**What is required.** Data is classified **attribute by attribute** and the privacy surface is kept deliberately small.

| Verifiable criterion | Citation |
|---|---|
| `user.username` and `sale.sold_by` (later `sold_by_username`) are **personal data**: restricted access, admissible in auditing, **not** in anonymous responses or public endpoints | §7 |
| `sale.sold_by_user_id` (T-12) is indirect personal data | §7 |
| `user.role` is internal confidential | §7 |
| `product`, `sale_item` and `category` carry no personal data | §7 |
| There is no end-customer personal data: the sale records the **operator**, not the buyer | §7 |
| The report is **not broken down by seller** (DP-02), and the index for that query is not prebuilt | §6.3, §7.1 |
| Classification does not depend on the column name: `sold_by` and `sold_by_username` are the same personal data | §7 |

---

### NFR-09 — Retention

| Data | Policy | Citation |
|---|---|---|
| Sales and their lines | Never deleted or edited. Indefinite | §7.1 |
| Products | Soft delete, never physical deletion | §7.1 |
| Categories and users | No deletion. If added for users, it must be restrictive (`FK-4`) | §7.1 |
| Image binary | Deleted on replacement or when the product is deactivated. It is the **only** data physically deleted | §7.1 |
| Password hash | No history or versioning | §7.1 |

**Mandatory order when deleting a binary:** first `image_key` is set to null and the transaction is committed; then the binary is deleted. **Atomicity between the database and storage is not promised.** An orphaned binary is harmless; a key pointing to a deleted binary is a permanently broken image (§7.1). The orphan cleanup policy is not defined (H-2, **open**).

---

### NFR-10 — Schema ownership and verifiability

**What is required.** A single source of truth for the schema, and that anyone can check that the documentation does not lie.

| Verifiable criterion | Citation |
|---|---|
| All DDL comes from **EF migrations**; no other piece of infrastructure defines schema | §3.2, §6.2, ADR-001 |
| **No column has a `DEFAULT`**: values are set by the domain | §3, §8 |
| Tables are singular inside the `sales` schema | §0 |
| If the model contradicts the engine, **the engine wins** and the document is broken | «Governed by», article X |
| Three reproducible queries (columns, constraints, indexes and extensions) check the document. Results of the **2026-09-19** measurement: **8 constraints** (5 `PK`, 2 `FK`, 1 `CHECK`) and **12 indexes**. **Those figures are no longer the expected ones today:** §13 (2026-09-20) considers `FK-3` and the unique `(sale_id, product_id)` done, which would raise them to 9 constraints and one more index. §10 was not repeated, so **it must be measured again** before fixing the new values (OBS-05) | §10, §13 D-2 |
| `pg_trgm` is installed by the **same migration** that creates the trigram index, not by `db/init/` | §6.2 |

---

### NFR-11 — Performance of the most frequent queries

**What is required.** Frequent queries must be backed by the index their access pattern needs, and none more.

| Pattern | Frequency | Backing the model requires | Citation |
|---|---|---|---|
| Q1 search product | High | `(category_id, name)` partial over active; trigrams over `name` partial over active (T-13) | §6.1, §6.2 |
| Q9 aggregated report | **High, the most costly** | Unique index `(sale_id, product_id)` with `INCLUDE (quantity, unit_price)`: aggregation does not touch the table | §6.2 |
| Q7 sales by range | High | `IX_sale_sold_at` already exists; the `DESC` direction is cosmetic on a single-column index | §6.2 |
| Q10 user by name | High | `IX_user_username` (unique) | §6.2 |

**The model's own caveat.** Index-only scans require an up-to-date visibility map; on an insert-only table, autovacuum fires rarely. **The mitigation is operational, not design** (§6.3).

**Not indexed, and justified:** loose `deleted_at`, `user.role`, `product.stock`, `sale.sold_by_user_id`, `product.image_key`, `user.password_hash` (§6.3).

---

### NFR-12 — Temporal consistency

| Verifiable criterion | Citation |
|---|---|
| All timestamps are `timestamptz`, without exception | §3 |
| The server runs in UTC | §3 |
| The only business instant of the system is `sale.sold_at` | §3, §8 |
| `product.deleted_at` is the only state transition tracked | §8 |

---

### NFR-13 — Maintainability and common language

| Verifiable criterion | Citation |
|---|---|
| Entities in English singular `PascalCase`; attributes in English singular `snake_case`; **never plural** in list attributes | §0 |
| Code and column names are in English | §1, article XI |
| The glossary is the project's single language | §12 |
| Objects EF generates keep their style (`PK_`, `IX_`, `FK_`); hand-written ones use `ck_{table}_{rule}` | §3.1 |
| Code collections remain plural (`Sale.Items`); the adapter translates | §0 |

---

## 3. What the model does NOT define

The model describes storage. It **sets no values** for the following quality attributes, and this document **does not invent them** [ASSUMPTION S-07]:

| Attribute | What is missing | What the model does have |
|---|---|---|
| Response times | No numeric target | Only which queries are frequent and which is the most costly (§6.1) |
| Volume and capacity | No figure for sales, products or concurrent users | **Current** volume: 5 categories, 1 user, 0 products, 0 sales, 0 lines (§10.4) |
| Availability and recovery | No RTO, RPO or windows | Nothing |
| Backups | No policy | Nothing |
| Observability (logs, metrics, traces) | No definition | Only what must **not** appear in logs (§7) |
| Accessibility and interface | No definition | The model mentions no client (S-09) |

**Why the current volume matters.** Several pending migrations —`sale_id NOT NULL`, `category_name NOT NULL`, `sold_by_user_id NOT NULL`— are **free today and expensive tomorrow**: as soon as the first sale exists, each needs a backfill (§10.4). It is an **execution order** requirement: those migrations are done before there is data.
