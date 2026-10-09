# User Stories — Simple Stock Flow

> **Single source:** [`spec/data-model.md`](../spec/data-model.md). Each story and criterion carries its citation («§2.3», `FK-2`, `D-04`…). Whatever does not come from the model is marked **[ASSUMPTION S-nn]** and explained in [`../assumptions.md`](../assumptions.md).
> **What this is:** the stories that the data model **makes necessary**: each exists because a table, a rule or an access pattern of the model would make no sense without it.
> **What it is not:** the `spec.md` of the original project. That document is not delivered (challenge README). These stories are reconstructed backwards from the model and may differ from it.

---

## 1. Actors

The model recognizes **a single type of person**: the **user**, «internal operator who authenticates and registers sales», with a role from a closed set of two (§1, §2.5). **There is no customer or buyer** (§1, §7).

| Actor | Source in the model | What is known about their permissions |
|---|---|---|
| **Administrator** (`admin`) | `user.role` | Creates sellers (DP-04, §13 D-3). The `admin` role is provisioned by the deployment from the environment, **nobody grants it at runtime** |
| **Seller** (`seller`) | `user.role` | Registers sales (§1). Without a token it gets 401, and with `seller` user registration gets 403 (§13 D-3) |
| **Deployment** | §9.2 | Provides the environment credentials with which startup creates the initial administrator |

> [ASSUMPTION S-03] **The model does not say which role may manage the catalog nor who may query sales and reports.** These stories assume: the administrator manages the catalog; both roles query catalog, sales and report; both roles register sales. It must be validated with the owner before implementing authorization.

---

## 2. Epics

| Epic | Title | Scope | Model entities |
|---|---|---|---|
| `EP-AUTH` | Identity and access | Log in, create sellers, initial administrator | `user` |
| `EP-CAT` | Catalog | Products, fixed categories, image, soft delete | `product`, `category` |
| `EP-SAL` | Sales | Register and query immutable sales | `sale`, `sale_item` |
| `EP-REP` | Report | Aggregation of sales by product and range | read model, no table (§1) |

---

## 3. Stories

### US-AUTH-001 — Log in {#US-AUTH-001}

**Epic:** `EP-AUTH`

> **As** an internal user (administrator or seller)
> **I want** to authenticate with my username and password
> **so that** I can access the operations my role allows

**Acceptance criteria:**

```gherkin
Scenario 1: Valid credentials
  Given a registered user named "ana" and her password
  When she authenticates with those credentials
  Then the system gives her a valid bearer credential
  And the response does not contain the password hash

Scenario 2: The name is normalized before lookup
  Given a user registered as "ana"
  When she authenticates typing "Ana " (capital letter and trailing space)
  Then the system recognizes her as "ana"
  # §2.5: without normalization, an «Ana » account could never log in

Scenario 3: Invalid credentials
  Given a nonexistent username or a wrong password
  When authentication is attempted
  Then the system rejects the attempt
  And the message does not reveal which of the two data failed [ASSUMPTION S-13]

Scenario 4: Protected operation without credential
  Given a request to a protected operation without a bearer credential
  When it reaches the API
  Then it answers 401
  # §13 D-3
```

| Field | Value |
|---|---|
| Access frequency | **High, at every login** (Q10, §6.1) |
| Citations | §1 «User», «Password hash»; §2.5; §6.1 Q10; §7 `password_hash`; §13 D-3 |
| Entities / rules | `user.username` unique and lowercase; `user.password_hash` never visible |
| Dependencies | US-AUTH-003 (at least one user must exist) |

---

### US-AUTH-002 — Seller registration by the administrator {#US-AUTH-002}

**Epic:** `EP-AUTH`

> **As** an administrator
> **I want** to register sellers
> **so that** they can register sales without sharing my account

```gherkin
Scenario 1: Successful registration
  Given an authenticated administrator
  When he creates a user with name "Luis " and a password
  Then the user is stored with name "luis" and role "seller"
  And the password is stored only as a hash produced by the hash port
  # §2.5, D-09

Scenario 2: Existing name
  Given a user "luis" already registered
  When the administrator tries to create "Luis "
  Then registration is rejected as a duplicate name after normalization
  # §2.5: uniqueness is enforced by the engine (IX_user_username)

Scenario 3: No credential
  Given a registration request without a bearer credential
  When it reaches the API
  Then it answers 401
  # defect A-1, closed (§13 D-3)

Scenario 4: Insufficient role
  Given an authenticated user with role "seller"
  When he tries to register a user
  Then it answers 403

Scenario 5: Nobody creates administrators at runtime
  Given any authenticated user, including an administrator
  When he tries to register a user with role "admin"
  Then the system rejects it
  # DP-04 (§11 H-3): the admin role is provisioned by the deployment

Scenario 6: Role outside the closed set
  Given a registration with a role other than "admin" or "seller"
  When it is processed
  Then the system rejects it
  # §2.5: `Roles.IsValid` (domain-only today; moves to the engine with T-20)
```

| Field | Value |
|---|---|
| Citations | §2.5; §9.2; §11 H-3 (DP-04); §13 D-3 |
| Rules | Lowercase and trimmed normalization (**domain-only**, T-20); uniqueness (**engine**); valid `role` (**domain-only**, T-20) |
| Dependencies | US-AUTH-001 |

---

### US-AUTH-003 — Provision the initial administrator {#US-AUTH-003}

**Epic:** `EP-AUTH`

> **As** the deployment owner
> **I want** startup to create the first administrator from environment credentials
> **so that** I can start the system without writing a secret in the repository

```gherkin
Scenario 1: Startup with no users
  Given a database with no users and administrator credentials in the environment
  When the application starts
  Then a user with role "admin" exists and its password is stored only as a hash

Scenario 2: Missing credentials
  Given a database with no users and no administrator credentials in the environment
  When the application starts
  Then the system does not invent default credentials [ASSUMPTION S-14]

Scenario 3: Repeated startup
  Given an administrator already created
  When the application starts again
  Then the user is neither duplicated nor modified
  # §10.4: today there is exactly one row in `user` [ASSUMPTION S-14: idempotency is inferred]
```

| Field | Value |
|---|---|
| Citations | §9.2; §10.4; D-09, D-10; article IX |
| Why it exists | Seeding the hash from SQL would require reimplementing the algorithm in SQL or versioning a literal hash; both options are discarded (§9.2) |
| Dependencies | None |

---

### US-CAT-001 — Create and modify a product {#US-CAT-001}

**Epic:** `EP-CAT`

> **As** an administrator [S-03]
> **I want** to create products and modify their name, price, stock and category
> **so that** I keep the catalog that is sold

```gherkin
Scenario 1: Valid creation
  Given an authenticated administrator
  When he creates a product with name "Martillo", price 25.50, stock 10 and an existing category
  Then the product is active with those five attributes
  And no additional attribute is stored
  # DP-03: name, price, stock, category and optional image, and nothing else

Scenario 2: Empty name
  Given a name made only of spaces
  When creating or renaming is attempted
  Then the system rejects it
  # §2.2: `Product.Rename` — «non-empty» is domain-only

Scenario 3: Name with spaces at the ends
  Given a name " Martillo "
  When it is saved
  Then it is stored as "Martillo"

Scenario 4: Non-positive price
  Given a price of 0 or negative
  When a product is created or its price changed
  Then the system rejects it
  # §2.2: `Product.ChangePrice`. A price 0 inserted via psql passes today (T-20 closes it)

Scenario 5: Price rounding
  Given a price 10.005
  When it is saved
  Then it is stored as 10.01
  # §2.2: 2 decimals, MidpointRounding.AwayFromZero, column numeric(18,2)

Scenario 6: Nonexistent category
  Given a category identifier that is not one of the five
  When the product is created or recategorized
  Then the system rejects it
  # FK-1

Scenario 7: Stock replenishment
  Given a product with stock 4
  When 6 units are replenished
  Then the stock is 10

Scenario 8: Concurrent modification
  Given two administrators who read the same product
  When both save a modification
  Then one of the saves is rejected due to a concurrency conflict
  # D-04, ADR-002 (xmin)
```

| Field | Value |
|---|---|
| Citations | §1 «Product»; §2.2; §3 `product`; §5 FK-1; DP-03; D-04 |
| Rules | `price > 0` (**domain-only**, T-20); `stock >= 0` (**engine**); category required (**engine**); non-empty name (**domain-only**) |
| Dependencies | US-AUTH-001, US-CAT-003 |

---

### US-CAT-002 — Search and query products {#US-CAT-002}

**Epic:** `EP-CAT`

> **As** an internal user
> **I want** to search products by part of the name and by category
> **so that** I quickly find what I am going to sell or review

```gherkin
Scenario 1: Partial text search
  Given active products "Martillo", "Martillo de goma" and "Taladro"
  When I search the text "marti"
  Then I get the two hammers, ordered by name

Scenario 2: Category filter
  Given products from several categories
  When I filter by "Herramientas"
  Then I only see products of that category

Scenario 3: Deactivated products do not appear
  Given a deactivated product
  When I search by any criterion
  Then it does not appear
  # Q1: «active»; global filter of ADR-003

Scenario 4: Pagination with total
  Given more products than the page size
  When I request the first page
  Then I receive that page and the total number of items
  # §6.1: Q1 implies an additional count query

Scenario 5: Product by identifier
  Given an identifier of an active product
  When I query it
  Then I receive its five attributes
```

| Field | Value |
|---|---|
| Access frequency | **High** (Q1, Q2, Q3; §6.1) |
| Citations | §6.1 Q1–Q3; §6.2 (T-13 indexes); ADR-003 |
| Dependencies | US-AUTH-001 |

---

### US-CAT-003 — List categories {#US-CAT-003}

**Epic:** `EP-CAT`

> **As** an internal user
> **I want** to see the available categories
> **so that** I choose one when creating a product or filtering a search

```gherkin
Scenario 1: Listing
  Given a freshly deployed system
  When I query the categories
  Then I see exactly five, ordered by name:
    "Electricidad", "Fontanería", "General", "Herramientas", "Pinturas"
  # §9.1

Scenario 2: Categories are not maintained
  Given any user, including an administrator
  When he tries to create, rename or delete a category
  Then the operation does not exist
  # §2.1, D-10: the repository is read-only
```

| Field | Value |
|---|---|
| Access frequency | High (Q4), Medium (Q5) |
| Citations | §1 «Category»; §2.1; §4.1; §9.1; D-10 |
| Review condition | If category maintenance is ever opened, accent- and case-sensitive uniqueness is reviewed **before** writing that CRUD (§4.1) |

---

### US-CAT-004 — Deactivate a product {#US-CAT-004}

**Epic:** `EP-CAT`

> **As** an administrator [S-03]
> **I want** to withdraw a product from the catalog without losing its past sales
> **so that** it stops being sold but the history stays intact

```gherkin
Scenario 1: Soft delete
  Given an active product with registered sales
  When the administrator deactivates it
  Then the product stops appearing in searches
  And its earlier sale lines keep showing its name and price of that time
  # §2.2, ADR-003, §7.1

Scenario 2: A deactivated product is not sold
  Given a deactivated product
  When a sale tries to include it
  Then the sale is rejected
  # §5: the line points to a product «not deactivated at the moment of the sale»

Scenario 3: Physical deletion fails loudly
  Given a product with sale lines
  When someone tries to delete it directly in the database
  Then the engine prevents it
  # FK-3 (RESTRICT)

Scenario 4: Its image is deleted
  Given a product with an image
  When it is deactivated
  Then `image_key` is set to null, the transaction is committed and then the binary is deleted
  # §7.1: the only data of the system that is physically deleted
```

| Field | Value |
|---|---|
| Citations | §2.2; §5 FK-3; §7.1; ADR-003; D-03 |
| Dependencies | US-CAT-001, US-CAT-005 |

---

### US-CAT-005 — Manage a product's image {#US-CAT-005}

**Epic:** `EP-CAT`

> **As** an administrator [S-03]
> **I want** to attach, replace or remove a product's image
> **so that** I identify it visually

```gherkin
Scenario 1: Attach
  Given a product without an image
  When an image is attached
  Then the binary goes to external storage and the product stores only its opaque key
  # §1: never the binary nor a path (D-08)

Scenario 2: No image
  Given a product without an image
  When it is queried
  Then `image_key` is NULL, never an empty string
  # §2.2

Scenario 3: Blank key
  Given a key made of spaces
  When attaching is attempted
  Then it is normalized to NULL

Scenario 4: Replace
  Given a product with an image
  When it is replaced
  Then the new key is committed in the database and then the previous binary is deleted
  # §7.1

Scenario 5: The second step fails
  Given the database committed the new key
  When deletion of the previous binary fails
  Then the operation is not rolled back and an orphaned binary remains
  # §7.1, H-2: it is harmless; no cleanup process is defined
```

| Field | Value |
|---|---|
| Citations | §1; §2.2; §3 `image_key varchar(512)` nullable; §7.1; D-08; H-2 |
| Dependencies | US-CAT-001 |

---

### US-SAL-001 — Register a sale {#US-SAL-001}

**Epic:** `EP-SAL`

> **As** an internal user
> **I want** to register a sale with several lines
> **so that** stock goes down and an unalterable record of what was sold remains

```gherkin
Scenario 1: Valid sale
  Given an authenticated seller and active products with enough stock
  When he registers a sale with two lines of different products
  Then the sale is attributed to that user with the current date and time
  And the stock of each product drops by the quantity sold
  And each line stores the product's name, price and category at that instant
  # §2.3, §2.4

Scenario 2: Withdrawal and line are a single operation
  Given a sale with a line whose product does not have enough stock
  When registration is attempted
  Then it is rejected entirely and no stock changes
  # §2.2 `Product.Withdraw`, §2.3 `Sale.AddItem`

Scenario 3: No lines
  Given a sale with no lines
  When confirmation is attempted
  Then it is rejected
  # §2.3 `Sale.EnsureConfirmable`

Scenario 4: Repeated product
  Given a sale with two lines of the same product
  When registration is attempted
  Then it is rejected
  # §2.3: the engine also prevents it with the unique (sale_id, product_id)

Scenario 5: Non-positive quantity
  Given a line with quantity 0 or negative
  When registration is attempted
  Then it is rejected
  # §2.4 `Quantity` (domain-only, T-20)

Scenario 6: Deactivated or nonexistent product
  Given a line pointing to a deactivated or nonexistent product
  When registration is attempted
  Then it is rejected
  # §5

Scenario 7: Simultaneous sales on the same product
  Given a product with stock 1 and two simultaneous sales of 1 unit
  When both are confirmed
  Then only one succeeds and stock never goes negative
  # D-04, ADR-002, `ck_product_stock_non_negative`

Scenario 8: Computed total
  Given a registered sale
  When it is queried
  Then the total and subtotals are computed on read and do not exist as columns
  # §1, article VII
```

| Field | Value |
|---|---|
| Access frequency | **High** (Q3 precedes the stock write; it is the contention point of D-04, §6.1) |
| Citations | §1 «Sale», «Sale line»; §2.3; §2.4; §3; §5; D-04; D-05 |
| Rules | At least one line (**domain-only**); product not repeated (**domain-only and engine**); single operation (**domain-only**); immutable (**domain-only**, by absence of an operation) |
| Pending | The explicit currency guard in `Sale.AddItem` is **T-05**; authorship with FK to `user` is **T-12** (§2.3, §5) |
| Dependencies | US-AUTH-001, US-CAT-001 |

---

### US-SAL-002 — Query a sale {#US-SAL-002}

**Epic:** `EP-SAL`

> **As** an internal user
> **I want** to see a sale with all its lines
> **so that** I check what was sold, to whom it is attributed and for how much

```gherkin
Scenario 1: Sale detail
  Given a registered sale
  When I query it by its identifier
  Then I see who registered it, when, and each line with product, quantity, unit price and subtotal
  And the sale total

Scenario 2: History does not follow the catalog
  Given a sale whose product was later renamed or changed price
  When I query it
  Then I see the name and price of the sale date
  # §1 «Frozen name»

Scenario 3: The sale is neither edited nor deleted
  Given a registered sale
  When someone tries to modify or delete it
  Then the operation does not exist
  # §2.3, §7.1
```

| Field | Value |
|---|---|
| Access frequency | Medium (Q6) |
| Citations | §1 «Frozen name»; §2.3; §6.1 Q6; §7 `sold_by` is personal data |
| Privacy | `sold_by` is personal data with restricted access and appears on receipts; it is not shown in anonymous responses (§7) |
| Dependencies | US-SAL-001 |

---

### US-SAL-003 — List sales by date range {#US-SAL-003}

**Epic:** `EP-SAL`

> **As** an internal user
> **I want** to list the sales of a period
> **so that** I review commercial activity

```gherkin
Scenario 1: Listing by range
  Given sales on several dates
  When I ask for those of a valid range
  Then I receive them ordered from newest to oldest, paginated and with the total

Scenario 2: Invalid range
  Given a range whose end precedes its start
  When I send it
  Then it is rejected
  # §1 «Date range»

Scenario 3: No unpaginated version
  Given any sales query
  When it is resolved
  Then it is always paginated
  # §6.1: Q8 is redundant and is removed from the port
```

| Field | Value |
|---|---|
| Access frequency | **High** (Q7) |
| Citations | §1 «Date range»; §6.1 Q7, Q8; §6.2 `IX_sale_sold_at` |
| Dependencies | US-SAL-001 |

---

### US-REP-001 — Sales report by product {#US-REP-001}

**Epic:** `EP-REP`

> **As** an internal user
> **I want** to see how much of each product was sold in a period
> **so that** I know what moves and what revenue it brings

```gherkin
Scenario 1: Aggregation by product
  Given sales in a range
  When I ask for the report of that range
  Then I receive a row per product with quantity sold and amount
  And the rows come ordered by amount descending
  # §6.1 Q9

Scenario 2: Invalid range
  Given a range with the end before the start
  When I send it
  Then it is rejected

Scenario 3: Recategorization within the range
  Given a product sold in September as "Herramientas" and in October as "Ferretería"
  When I ask for the report from September to October
  Then two rows appear for that product, one per frozen label
  # §11.1 (H-1, closed 2026-09-20): grouped by product_id, product_name, category_name

Scenario 4: A closed report does not change
  Given a report of an already finished range
  When a sale with a new label is later registered outside that range
  Then the result of the original report does not change
  # §11.1: «a closed report must never change»

Scenario 5: No personal data
  Given any report
  When it is returned
  Then it includes neither seller nor any personal data
  # DP-02 (§7.1)

Scenario 6: Not persisted
  Given a report requested twice
  When it is resolved
  Then it is computed in the engine each time; no report table exists
  # §1, §2, D-06
```

| Field | Value |
|---|---|
| Access frequency | **High. The most costly query** (Q9) |
| Citations | §1 «Sales report»; §6.1 Q9; §6.2; §7.1; §11.1 H-1; DP-02; D-06; ADR-004 |
| **Conflict declared by the model itself** | `spec.md` CA-06.1 says «one row per product». With H-1 there can be more than one row per product if there was a recategorization. The model marks the rewrite of CA-06.1 as an **owner decision pending** (§11.1). These stories follow decision H-1, which is later |
| Related defect | A-7 / DP-01: the product name currently uses a tie-break that H-1 rejects; **it is reported, not fixed** (§11.1) |
| Dependencies | US-SAL-001 |

---

## 4. What the model does NOT justify

These capabilities have **no story** because the model discards them or does not back them. If someone asks for them, it is new scope.

| Capability | Why not | Citation |
|---|---|---|
| Create, rename or delete categories | They are seed data, no maintenance | §2.1, D-10 |
| Edit or delete a sale | It is an accomplished, immutable fact | §1, §2.3 |
| Manage customers or buyers | There is no customer entity | §1, §7 |
| Additional product attributes (description, SKU…) | Closed by DP-03 | §1 |
| More than one currency | Single-currency by construction | D-05 |
| Report by seller | Decided: not broken down | DP-02, §7.1 |
| Change auditing (`created_at`/`updated_at`) | Closed decision, no requirement | §8 |
| Export or external analytics | Does not exist | §7.1 |
| Granting the `admin` role at runtime | The deployment provisions it | DP-04, §13 D-3 |
| Payments, cards, health data | Do not apply | §7 |
