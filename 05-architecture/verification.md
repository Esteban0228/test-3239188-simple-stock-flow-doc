# Closing Verification — Architecture against 01–04 and against the model

> **Step 6 of the challenge:** go back to the architecture and check that it fits everything before it and the model.
> **Single source:** [`spec/data-model.md`](../spec/data-model.md). Assumptions in [`../assumptions.md`](../assumptions.md).
> This document has three parts: **what was checked and how**, **the mismatches the model itself contains** and **what was corrected** in the first pass of [`overview.md`](overview.md).

---

## 1. Method

1. Each section of the model (§0–§13) must land in at least one document of 01–05 (backward coverage, §2).
2. Each access pattern Q1–Q10 must have a story and a port (§3).
3. Each story must reach an aggregate and its rules (§4).
4. Cross references were checked **mechanically**: stories, rules, assumptions and relative links (§5).
5. What the model states contradictorily is not resolved silently: it is recorded (§6).

---

## 2. Coverage of the model's sections

| Model section | Where it lands |
|---|---|
| §0 Naming convention | `05` overview §8; `04` NFR-13 |
| §1 Glossary | `02` glossary; `02` entities §3 |
| §2 Entities and invariants | `02` entities; `05` overview §4 and §6 |
| §3 Physical model | `05` overview §8; `04` NFR-05, NFR-10, NFR-12 |
| §3.1 Object naming convention | `05` overview §8; `04` NFR-13 |
| §3.2 Migrations | `05` overview §7 and §8; `04` NFR-10 |
| §4 Constraints and indexes | `05` overview §6; `02` entities §7; `04` NFR-01 |
| §4.1 Uniqueness of `category.name` | `02` entities §2.1; `04` US-CAT-003; `01` scope |
| §5 Foreign keys | `05` overview §4 and §6; `02` entities §5 |
| §6 Access patterns and indexes | `05` overview §5; `04` NFR-11; `04` stories |
| §7 Privacy and retention | `04` NFR-06, NFR-08, NFR-09; `05` overview §9 |
| §8 No auditing | `01` scope; `02` events; `03` vision |
| §9 Seed | `04` US-AUTH-003, US-CAT-003; `05` overview §8 |
| §10 Verification | `04` NFR-10; this document |
| §11 Gaps | `01` scope §3; `03` problem §8; `04` US-REP-001 |
| §12 Signature and exclusions | `05` overview §10; `01` scope §2 |
| §13 Declared debt | `03` vision §4; `04` NFR-01; this document §6 |

---

## 3. Coverage of the access patterns

| Pattern | Frequency | Story | Port | Index backing it |
|---|---|---|---|---|
| Q1 Search product | High | US-CAT-002 | Product repository | `(category_id, name)` partial + trigrams (T-13, pending) |
| Q2 Product by identifier | High | US-CAT-002 | Product repository | Primary key |
| Q3 Products by batch | High | US-SAL-001 | Product repository | Primary key |
| Q4 List categories | High | US-CAT-003 | Category repository | `IX_category_name` |
| Q5 Category by identifier | Medium | US-CAT-001 | Category repository | Primary key |
| Q6 Sale with its lines | Medium | US-SAL-002 | Sale repository | Primary key; `(sale_id, product_id)` |
| Q7 Sales by range | High | US-SAL-003 | Sale repository | `IX_sale_sold_at` |
| Q8 Sales by range unpaginated | Low | **none: removed** | — | — |
| Q9 Aggregated report | High, the most costly | US-REP-001 | Read port | `(sale_id, product_id)` with `INCLUDE` |
| Q10 User by name | High | US-AUTH-001 | User repository | `IX_user_username` |

**Q8 without consumer.** The model asks to remove it from the port (§6.1). The architecture does not include it: it is a **design decision**, not an oversight.

---

## 4. Traceability from stories to aggregates and rules

| Story | Aggregate / piece | Domain rules | Ports |
|---|---|---|---|
| US-AUTH-001 | `User` | BR-USR-01, BR-USR-02, BR-USR-05 | User repository, hash |
| US-AUTH-002 | `User` | BR-USR-01 to BR-USR-04, BR-USR-06 | User repository, hash |
| US-AUTH-003 | `User` | BR-USR-03, BR-USR-05, BR-USR-06 | User repository, hash |
| US-CAT-001 | `Product` | BR-PRO-01, 02, 03, 05, 08, 09 | Product and category repositories |
| US-CAT-002 | `Product` (read) | BR-PRO-07 (active only) | Product repository |
| US-CAT-003 | `Category` | BR-CAT-02, BR-CAT-03 | Category repository |
| US-CAT-004 | `Product` | BR-PRO-07, BR-LIN-01, BR-GEN-02 | Product repository, images |
| US-CAT-005 | `Product` | BR-PRO-06 | Product repository, images |
| US-SAL-001 | `Sale`, `SaleItem`, `Product` | BR-SAL-01 to 06, BR-LIN-01 to 06, BR-PRO-03, BR-PRO-04 | Sale and product repositories |
| US-SAL-002 | `Sale` | BR-SAL-05, BR-LIN-03, BR-LIN-04 | Sale repository |
| US-SAL-003 | `Sale` (read) | BR-REP-01 | Sale repository |
| US-REP-001 | Read model | BR-REP-01 to BR-REP-04 | Read port |

---

## 5. Mechanical checks

Run on the files of this delivery.

| Check | Result |
|---|---|
| Every `US-…` story cited in 01–05 exists in `user-stories.md` | see §7 |
| Every `BR-…` rule cited in 01–05 exists in `entities-and-rules.md` | see §7 |
| Every `S-nn` cited is defined in `assumptions.md` and vice versa | see §7 |
| Every relative link points to an existing file (except `spec/data-model.md`, which is already in the repository) | see §7 |

---

## 6. Mismatches the model itself contains

When contrasting §3 against §10 and §13, contradictions appear **inside the model**. None changes the architecture, but whoever implements must know them. This delivery **does not resolve them**: it records them and says which one was followed.

| # | Mismatch | Where | Treatment in this delivery |
|---|---|---|---|
| OBS-01 | The model says «22 columns» in the title of §3 and in §12, but §3 (note on `xmin`) and §10.1 speak of **21**. The §10.1 measurement (2026-09-19) returns 21 and clarifies that `xmin` does not count. **Most likely reading (inference, not a fact of the model):** 22 = the 21 measured + `product.deleted_at`, which §3 already marks as **engine** (T-09) and §13 D-1 settles on 2026-09-20; the title was updated and the measurement was not. By that criterion, `sold_by_user_id` (T-12) and `sale_item.category_name` (T-11, pending in §3) would be outside the count, and with them the total would reach **24**. The internal link of §8 to `#3-modelo-físico--las-21-columnas` confirms the title changed | §3, §8, §10.1, §12, §13 D-1 | No total figure is stated as a measurement. **21** is cited as what was measured (2026-09-19) and **22** as what the title declares, with the explanation above marked as a reading |
| OBS-02 | The `product` table of §3 includes a `category_name` row whose note speaks of the label frozen «at the moment of the sale» and of the foreign key being avoided. §2.4 and §10.1 place it in `sale_item` | §3, §2.4 | Treated as a **misplaced row**: `category_name` belongs only to `sale_item` |
| OBS-03 | §6.2 marks the unique index `(sale_id, product_id)` as «missing (T-13)». §2.3, §4 and §13 D-2 say it **exists** since T-20. §10.3 (2026-09-19) treats it as absent | §6.2, §2.3, §4, §10.3, §13 | §13 prevails (2026-09-20, later). T-13 remains pending for the other two indexes and for dropping `IX_sale_item_sale_id` |
| OBS-04 | §4 and §5 speak of «eight constraints» and «today two exist» foreign keys. §4 and §13 D-2 mark `FK-3` as **engine** | §4, §5, §10.2, §13 | The count of §4 and §5 is that of 2026-09-19. With `FK-3`, foreign keys would be three |
| OBS-05 | **Different dates in the same document.** The §10 outputs are from 2026-09-19. §13 was measured on 2026-09-20 and reclassifies marks (T-09, T-20). The body was not fully rewritten | §10, §13 | The updated marks of §13 prevail. The literal outputs of §10 **no longer match the engine** and would need to be repeated |
| OBS-06 | **Contradictory status of T-11.** §2.4 marks `sale_item.category_name` as **engine (T-11)**. §3 marks it **pending (T-11)** and §10.1 treats it as absent. §13 does not mention it | §2.4, §3, §10.1, §13 | Its status is not stated. The contradiction is recorded |
| OBS-07 | **Partial status of T-20.** §13 declares three pieces settled (`sale_id NOT NULL`, composite unique, `FK-3`). The five `CHECK`s (`price > 0`, `quantity > 0`, etc.) remain marked «domain-only · T-20» in §2 and §4 | §2, §4, §13 | Assumed **not pushed down** [ASSUMPTION S-10] |
| OBS-08 | `spec.md` CA-06.1 says «one row per product»; decision H-1 produces more than one if there was a recategorization. **The model itself points it out** and leaves the rewrite to the owner | §11.1 | H-1 is followed. Owner decision pending |
| OBS-09 | The report's product name currently uses the tie-break that H-1 rejects (DP-01, defect A-7). **It is reported, not fixed** by the owner's instruction | §11.1 | Recorded in US-REP-001 |

> **What the whole teaches.** The model declares itself signed and «no agent changes its marks on its own» (§13). Mismatches OBS-01 to OBS-06 are the consequence of having corrected §13 without rewriting §3, §4, §5 and §10. That is exactly the kind of «lie» the document itself calls debt.

---

## 7. Result of the mechanical checks

Run with a script over the files of this delivery:

| Check | Result |
|---|---|
| Stories defined in `user-stories.md` | **12** (US-AUTH 3, US-CAT 5, US-SAL 3, US-REP 1) |
| Stories cited in other documents without being defined | **none** |
| Stories defined that no other document cites | **none** |
| `BR-…` rules defined in `entities-and-rules.md` | **39** |
| Rules cited without being defined | **none** |
| Assumptions defined / cited | **18 / 18**, no gaps or repeats |
| Broken relative links | **none** |

**Note on rules.** Some rules (`BR-GEN-…`, `BR-LIN-05`, `BR-LIN-06`…) are not cited by identifier in `04` and `05` because they are cited there by range («BR-SAL-01 to 06») or covered from the non-functional requirements. It is not an omission: the table of §4 covers them.

---

## 8. Corrections applied to the first pass of `overview.md` and to the final review

| # | Correction | Reason |
|---|---|---|
| 1 | The rule count of §6 went from 27 to 25 and, in the final review, to **26** | The table was recounted row by row with a script: it has 26 rules. The two earlier figures were incorrect |
| 2 | A reference to a folder (`09`) that this challenge does not write was removed | Out of the challenge's scope |
| 3 | Assumptions were renumbered contiguously (S-01 to S-18) and unified in `assumptions.md` | There were gaps and a repeated identifier |
| 4 | OBS-01 was rewritten: the model's «22» is explained as 21 measured + `deleted_at`, marked as a reading and not as a fact | The earlier version said 22 «matches no count», and it does have a plausible explanation within the model itself |
| 5 | NFR-10 no longer presents «8 constraints and 12 indexes» as the current expected result | Those figures are from 2026-09-19; §13 makes them obsolete (OBS-05) |
| 6 | The columns note of `01-context/overview.md` points to OBS-01 with the same explanation | Keep a single version of the figure across all documents |
| 7 | The whole documentation was translated to English | English documentation was requested by the student's course requirement (it is also the language convention of the team's ADR-001); identifiers `RN-`, `HU-`, `RNF-` became `BR-`, `US-`, `NFR-` |
