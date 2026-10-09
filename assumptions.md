# Assumptions Register — Simple Stock Flow

> **Challenge rule (README, rule 3):** every statement must be traceable to the data model; **if something does not come from the model, it is marked as an assumption**.
> This register lists **all** the assumptions in documents 01–05, where they are used and how to validate them. In the documents they appear as **[ASSUMPTION S-nn]**.
> Source: [`spec/data-model.md`](spec/data-model.md). An assumption **is not a fact**: it is a reasonable reading that must be confirmed by whoever holds the data.

---

## 1. Assumptions

| ID | Assumption | Why it had to be assumed | Where it is used | How to validate it |
|---|---|---|---|---|
| **S-01** | The documentation is written in **English**; code and column identifiers are also in English | The model and the challenge README are in Spanish; the model's glossary requires code and columns in English (§1, article XI). The model does not regulate the documentation language. The team's own convention (ADR-001 of the LexIA docs) is English for technical documentation | All | Confirm with the owner |
| **S-02** | The system is **a single deployable** | The model does not state how many processes exist. It is inferred from the single `sales` schema that «groups the entire system» (§3) and the single API repository (§12) | `05` overview, `02` events | Confirm the deployment style |
| **S-03** | The **administrator** manages the catalog; **both roles** query catalog, sales and report and register sales | The model only says what each role does in two places: the `seller` registers sales and the `admin` creates sellers (§1, §9.2, DP-04). It does not say who manages the catalog or who sees reports | `04` stories, `01` overview and scope, `03` problem | **Priority.** Define the permissions matrix before implementing authorization |
| **S-04** | The business is a **hardware store** | Inferred only from the five seeded categories (§9.1). The model does not name the sector | `01`, `03` | Confirm the sector. The problem does not depend on this assumption |
| **S-05** | The **High / Medium** priority of each NFR | The model assigns no priorities; they were read from the weight it gives to each attribute | `04` NFR | Have the owner confirm or change them |
| **S-06** | The **business facts** that could be domain events | The model **defines no events**; they were derived from the operations it names per aggregate | `02` events | Only needed if a real consumer appears |
| **S-07** | **No targets** are set for performance, availability, volume, backups, observability or other environments | The model gives no figures; none are invented | `04` NFR §3, `01` scope, `05` | Ask the owner for targets |
| **S-08** | Authentication uses a **bearer credential** (token) | Inferred only from «no token → 401» (§13 D-3). The model names neither the mechanism nor the session duration | `04` US-AUTH, NFR-07, `05` | Review `api-contract.md` (not delivered) |
| **S-09** | The **client** (interface) is not defined | The model describes storage, not any client | `05` component map, `04` NFR §3 | Confirm whether there is an interface and of what kind |
| **S-10** | The **five `CHECK`s** of T-20 (`price > 0`, `quantity > 0`, non-empty category name, `role`, lowercase name) are assumed **not yet pushed down** to the engine | §4 marks them «domain-only · T-20»; §13 declares as settled only `sale_id NOT NULL`, the composite unique index and `FK-3` | `04` NFR-01, `03` vision | **Measure against the engine** with the §10 queries |
| **S-11** | ID generation and the sale timestamp are ports or domain services | The model says the domain sets them, not how (§3, §5) | `05` §5.2 | Decide during implementation |
| **S-12** | The **transaction** of «withdraw stock and add the line» is opened by the use case | The model says it is a single operation (§2.3) but not where the transaction is opened | `05` §6 | Review `plan.md` (not delivered) |
| **S-13** | The message of a failed login **does not reveal** which datum failed | Good practice; the model does not mention it | `04` US-AUTH-001 | Confirm or withdraw |
| **S-14** | Startup **does not invent default credentials** and is **idempotent** | The model says startup creates the administrator from environment credentials and that today there is one row (§9.2, §10.4) | `04` US-AUTH-003 | Confirm behavior with missing credentials and repeated startups |
| **S-15** | The users' working phrases (JTBD) | There are no interviews or usage descriptions in the material | `03` problem | Validate with whoever operates the shop |
| **S-16** | The user's **current solution is not described** | The model does not say how inventory is handled today | `03` problem | Research with the user |
| **S-17** | The **success metrics** and **key results** are our reading | The model defines no product metrics or goals | `03` problem and vision | Have the owner confirm them |
| **S-18** | The external image storage is an **object store** | The model only says «external storage» and «opaque key» (§1, D-08); it names neither type nor provider | `01` scope | Confirm provider and type |

---

## 2. What the model does not deliver and could not be reconstructed

| What | Where the model cites it | Consequence |
|---|---|---|
| `constitution.md`, `spec.md`, `plan.md`, `architecture.md`, `tasks.md`, `api-contract.md`, `adr/`, `traspaso/HANDOFF-TECNICO.md` | «Governed by», §12, §11.1 | Cited **only** for what the model says about them |
| Content of `D-01` and `D-02` | «Governed by» | Cannot be reconstructed |
| Detail of A-1 (beyond «user registration was anonymous») | §9.2, §13 D-3 | Only what the model states is used |
| Real status of T-05, T-10, T-11 and T-13 | §2–§6 | The model's reading and its internal contradiction are reported (see [`05-architecture/verification.md`](05-architecture/verification.md)) |

---

## 3. Assumptions to validate first

Ordered by how much they would change the result if wrong:

1. **S-03** — permissions per role: changes the authorization stories and NFR-07.
2. **S-10** — status of the five `CHECK`s: changes which rules belong to the engine and which remain debt.
3. **S-02** — single deployable: changes the architectural style.
4. **S-08** — authentication mechanism: changes the security port.
