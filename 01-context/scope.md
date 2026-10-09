# System Scope — Simple Stock Flow

> **Single source:** [`spec/data-model.md`](../spec/data-model.md). The scope is drawn **with what the model closes**: §12 expressly lists what is left out and states that «the three [decisions] are decided and are not reopened».
> **Assumption** = something that does not come from the model: **[ASSUMPTION S-nn]**, in [`../assumptions.md`](../assumptions.md).
> Scope prevents uncontrolled growth: defining what the system does **not** do is as important as defining what it does.

---

## 1. In scope

### Features

| # | Feature | Description | Aggregate | Citation |
|---|---|---|---|---|
| 1 | Authentication | Internal users log in with username and password; without credential, 401 | `User` | §2.5, §13 D-3 |
| 2 | Seller registration | The administrator creates sellers; the `admin` role is not granted at runtime | `User` | §9.2, DP-04 |
| 3 | Initial administrator | Startup creates it from environment credentials | `User` | §9.2 |
| 4 | Product catalog | Create, change price and stock, recategorize and deactivate products with **five attributes** | `Product` | §2.2, DP-03 |
| 5 | Product search | By partial text and category, active only, ordered by name and paginated | `Product` | §6.1 Q1–Q3 |
| 6 | Categories | Query the five fixed categories | `Category` | §2.1, D-10 |
| 7 | Product image | Attach, replace or remove; opaque key, binary in external storage | `Product` | §1, §7.1, D-08 |
| 8 | Sale registration | Sales of one or more lines; withdraw stock; immutable | `Sale` | §2.3, §2.4 |
| 9 | Sale queries | One sale with its lines; list by range, paginated | `Sale` | §6.1 Q6, Q7 |
| 10 | Sales report | Aggregation by product over a range; not persisted | read model | §1, §6.1 Q9, §11.1 |

The user stories that develop each one are in [`../04-requirements/user-stories.md`](../04-requirements/user-stories.md).

### Included integrations

| External system | Type | Purpose | Citation |
|---|---|---|---|
| **PostgreSQL 16** | Database | Storage of the five tables of the `sales` schema | «Verified against» |
| **External storage** | Object store [ASSUMPTION S-18: the model only says «external storage»] | Product image binaries, referenced by opaque key | §1, §7.1, D-08 |

> **Storage provider: not defined in the model.** None is named.

### Environments

The model mentions only the environment where it was verified: a PostgreSQL container (`simple-stock-flow-db-1`) started with `docker compose` from the infrastructure repository, with the server in **UTC** (§3, §10). **Other environments are not defined** [ASSUMPTION S-07].

---

## 2. Out of scope

What the system **does not build** and why. Everything is a **closed decision** unless stated otherwise.

| # | Out of scope | Reason | Future? | Citation |
|---|---|---|---|---|
| 1 | Customer / buyer entity | Does not exist: the sale records the operator, not the buyer | Undefined | §1, §7 |
| 2 | Multi-currency sales | The system is single-currency by construction; not reintroduced | No | D-05 |
| 3 | Additional product attributes (description, SKU, reference code) | Closed by DP-03 | No | §1, §12 |
| 4 | Report broken down by seller | DP-02; it would cross personal data | No | §6.3, §7.1 |
| 5 | Change auditing (`created_at`, `updated_at`, change log) | No requirement; closed decision | **Only if a real audit question appears**; then it returns to the owner | §8 |
| 6 | Category maintenance (create, rename, delete) | They are read-only seed data | **Explicit review condition:** if opened, accent/case uniqueness is reviewed first | §4.1, D-10 |
| 7 | Editing or deleting sales | Accomplished, immutable fact | No | §2.3 |
| 8 | Physical deletion of products | Soft delete | No | §2.2, ADR-003 |
| 9 | Granting the `admin` role at runtime | DP-04: the deployment provisions it | No | §11 H-3 |
| 10 | Payments and cards; health data | Do not apply | No | §7 |
| 11 | External analytics and export | Do not exist; the report exposes no personal data | Undefined | §7.1 |
| 12 | Anonymization for analytics | Conscious omission: there is no analytics | Undefined | §7.1 |
| 13 | API contract, business requirements and technical decisions of the original project | Live in documents that are **not delivered** | — | §12 |

### What another system or team handles

| Feature | Who | Why not us |
|---|---|---|
| Storage of image binaries | External storage | The system only stores an opaque key; the binary does not take part in the database transaction (§7.1) |
| Production of the password hash | A hash port (security adapter) | The domain must never see the plaintext password (§2.5, D-09) |
| Provisioning of the `admin` role | The deployment, from the environment | No user grants it at runtime (DP-04) |

---

## 3. Scope decisions still open

| # | Topic | Status | Citation |
|---|---|---|---|
| H-2 | Cleanup policy for orphaned image binaries | **Open**; operational, no impact on the deliverable | §11 |
| CA-06.1 | Rewrite the `spec.md` criterion «one row per product» so it does not contradict H-1 | **Owner decision pending** | §11.1 |
| — | Which role may manage the catalog, query sales and report | **Not defined** [S-03] | — |
| — | Expected data and user volume | **Not defined** | §10.4 |

---

## 4. Criteria to accept a scope change

What the model itself uses to decide what gets in:

1. **Is there a requirement asking for it?** No requirement, no scope (§8, DP-03).
2. **Does any query or port need it?** No access pattern, no index or column (§6, §8).
3. **Does it contradict a closed decision?** Then it goes back to its owner; it is not resolved with code (§8, §11.1).
