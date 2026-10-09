# Glossary — Simple Stock Flow

> **Single source:** [`spec/data-model.md`](../spec/data-model.md) §1 (domain glossary) and the terms the model uses with a meaning of their own. It is the project's single language (§12).
> Code and column names are in English (article XI, §1); business terms are explained in English too. Seed data values (category names) stay in Spanish.
> If a term is not here and is used, it is added with its definition and citation.

---

## 1. Business terms

| Term | Definition | Where it lives (technical) | Citation |
|---|---|---|---|
| **Product** | Catalog item. Has name, price, stock, category and optional image, **and nothing else** | `Product` · table `product` | §1, DP-03 |
| **Category** | Classification a product belongs to. **Fixed set of five**, seeded, no maintenance | `Category` · table `category` | §1, D-10 |
| **Price** | Current monetary value of the product in the catalog. Strictly positive | `Money` · `product.price` | §1 |
| **Stock** | Available units of the product. Never negative | `product.stock` | §1 |
| **Product image** | **Opaque key** of the binary in external storage; neither the binary nor a path. Absent is represented by `NULL`, never by an empty string | `product.image_key` | §1, D-08 |
| **Sale** | Accomplished and **immutable** commercial fact: who, when and what. Once registered it is neither edited nor deleted | `Sale` · table `sale` | §1 |
| **Sale line** | Row of the sale: product, quantity and **frozen price** of the moment. Does not exist outside its sale | `SaleItem` · table `sale_item` | §1 |
| **Quantity** | Units sold in a line. Strictly positive | `Quantity` · `sale_item.quantity` | §1 |
| **Sale total** | Sum of subtotals. **Computed, not stored** | `Sale.Total` · no column | §1 |
| **Line subtotal** | Unit price times quantity. **Computed, not stored** | `SaleItem.Subtotal` · no column | §1 |
| **User** | Internal operator who authenticates and registers sales. **There is no customer or buyer entity** | `User` · table `user` | §1 |
| **Role** | Attribution of the user within a closed set of two: `admin` or `seller` | `user.role` | §1 |
| **Password hash** | Irreversible fingerprint of the password. The domain **never sees the plaintext** | `user.password_hash` | §1, D-09 |
| **Date range** | Time window of the report. The end cannot precede the start | Application-layer value object · no table | §1 |
| **Sales report** | Aggregation by product over a range. **Not persisted**: computed in the engine through a read port | Read model · no table | §1, D-06 |
| **Frozen value** | Copy of the value (price, product name, category name) that the line stores **at the moment of the sale** and that does not follow the catalog. It is not denormalization: it is a fact of the sale itself | `sale_item.product_name`, `unit_price`, `category_name` | §1, §2.4 |

### Terms the model discards

| Term | Status | Citation |
|---|---|---|
| **Sale currency** | Does not exist. The system is single-currency; no table has a currency column | §1, D-05 |
| **Customer / buyer** | Does not exist as entity or datum | §1, §7 |
| **Description, SKU, reference code** (of the product) | Closed by DP-03 | §1 |

---

## 2. Terms from the model's vocabulary

What a reader needs to understand the model without opening the documents that are not delivered.

| Term | Meaning | Citation |
|---|---|---|
| **Engine** | Rule mark: exists in Postgres right now. A manual `INSERT` respects it or fails | «How to read» |
| **Domain-only** | Rule mark: guaranteed by application code and nothing else. An `INSERT` through `psql` skips it silently | «How to read» |
| **Pending (T-xx)** | Rule mark: does not exist yet; that task adds it | «How to read» |
| **Invariant** | Rule that must always hold, not only when someone remembers | §2 |
| **Aggregate root** | Entity through which a group of entities treated as a unit is accessed | §2 |
| **Aggregate** | Group of entities treated as a consistency unit (`Product`, `Sale` with its lines, `User`) | §2 |
| **Port** | Contract through which the domain asks something from the outside or exposes a capability (hash, report read, repositories) | §1, §2.5 |
| **Adapter** | Piece that implements a port against concrete technology (for example, persistence) | §0, «How to read» |
| **Last-resort barrier** | Engine constraint that exists so a violation fails loudly (`FK-3`, the stock `CHECK`) | §5, ADR-002, ADR-003 |
| **Soft delete** | Marking a product as deactivated (`deleted_at`) instead of deleting it | §2.2, ADR-003 |
| **Shadow property** | Column the mapping knows but the aggregate does not expose (`deleted_at`, `xmin`) | §3, D-03 |
| **Seed** | Data born with the schema (the five categories) | §9 |
| **Optimistic concurrency** | Detecting conflict on save instead of locking on read; the witness is `xmin` | §3, D-04, ADR-002 |
| **Declared debt** | Discrepancy between the document and the engine that the document acknowledges in writing, with status `Open` or `Settled` | §13 |

### Reference codes the model uses

| Code | What it is | Where it is detailed |
|---|---|---|
| `D-01`…`D-10` | Technical decisions | `plan.md` §1 (**not delivered**); the model gives the content of D-03…D-10 |
| `DP-01`…`DP-04` | Product decisions | DP-01 (tie-break of the product name), DP-02 (no report by seller), DP-03 (five attributes), DP-04 (nobody grants `admin`) |
| `ADR-001`…`ADR-004` | Structural decisions | `adr/` (**not delivered**) |
| `FK-1`…`FK-4` | Foreign keys of the policy | §5 |
| `H-1`…`H-3` | Gaps with an owner | §11 |
| `A-1`, `A-7` | Measured API defects. A-1: user registration was anonymous (closed). A-7: tie-break of the product name | A-7 is cited in `traspaso/HANDOFF-TECNICO.md` §6.1 (**not delivered**); the model cites no source for A-1 |
| `T-02`…`T-20` | Tasks | `tasks.md` (**not delivered**) |
| Articles `VII`, `IX`, `X`, `XI` | Principles of the constitution | `constitution.md` (**not delivered**) |

---

## 3. Usage rules

1. Before using a business term in code, documentation or conversation, this glossary is consulted.
2. If the term is not defined, it is added with its definition and citation.
3. If there is disagreement, **the model wins**; and if the model contradicts the engine, **the engine wins** (article X).
