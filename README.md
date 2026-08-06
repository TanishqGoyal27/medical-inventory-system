# Centralized Healthcare Inventory & Prescription Management System

A database system for Punjab government dispensaries, designed to address common
healthcare-facility problems: expired medicine distribution, poor inventory control,
prescription misuse, and fragmented record-keeping. The system models patients, staff
(doctors/pharmacists), medicine inventory, suppliers, prescriptions, and transaction
logs in a single normalized Oracle database, with business logic enforced at the
database layer via triggers, procedures, and functions.

Built as a course project for **UCS310 — Database Management Systems**,
Thapar Institute of Engineering & Technology.

**Team:** Ridhi, Tanishq Goyal, Saanjal Jain
**Instructor:** Dr. Simranjit Kaur

---

## What's in this repo

| File | What it is |
|---|---|
| `healthcare_inventory_system_final.sql` | Full backend — schema, data, triggers, procedures, functions, views, indexes, and required queries. This is the core deliverable. |
| `healthcare_inventory_redesigned.html` | A static front-end **UI/UX prototype** to visualize how the system's screens and workflows would look. It uses hardcoded sample data and is **not connected to the Oracle database** — no live queries, no backend calls. Built purely to demonstrate the intended user experience for admins, doctors, and pharmacists. |
| `dbmsreport_*.pdf` | Course project report/documentation. |

---

## Database design

- **17 normalized tables**, including an ISA (supertype/subtype) hierarchy —
  `Staff` splits into `Doctor` and `Pharmacist` — plus core entities like `Patient`,
  `Medicine`, `Medicine_Inventory`, `Prescription`, `Prescription_Items`,
  `Transaction_Log`, `Supplier`, `Dispensary`, and `Inventory_Audit`.
- Enforced via primary/foreign key constraints, `NOT NULL` checks, and sequences
  for surrogate keys.

## Business logic (triggers, procedures, functions)

- **`trg_isa_role_check`** — auto-creates the correct `Doctor`/`Pharmacist` subtype
  row when a new `Staff` record is inserted.
- **`trg_no_expired_issue`** — blocks any transaction attempting to issue an
  expired medicine batch.
- **`trg_pharmacist_only_issue`** — role-based access control: only staff with the
  `PHARMACIST` role can issue medicine.
- **`trg_update_stock`** — keeps `Medicine_Inventory` quantities in sync across
  stock-in, issue, return, expired-removal, and adjustment transactions.
- **`trg_audit_inventory`** — full audit trail logging quantity-before/after and
  the acting staff member for every inventory transaction.
- **`dispense_prescription`** — a transactional procedure that fulfills a
  prescription line-by-line: picks stock using earliest-expiry-first (FIFO),
  handles partial fulfillment when stock is insufficient, logs each issuance as a
  transaction, and rolls back cleanly on error.
- **`get_patient_history`** — cursor-driven report of a patient's full visit and
  prescription history.
- **`fn_patient_age`** / **`fn_is_medicine_available`** — derived-value helper
  functions used across queries and reports.

## Views

- `v_low_stock` — medicines below reorder threshold, non-expired.
- `v_expiry_alerts` — batches expiring within 60 days.
- `v_patient_prescription_summary` — multi-table rollup of prescriptions per patient.
- `v_dispensary_stock_summary` — per-dispensary stock value and alert counts.

## Required queries

Includes examples of multi-table joins, nested subqueries, correlated subqueries,
`GROUP BY`/`HAVING`, and view/function usage — covering the core query patterns
from the course syllabus.

---

## How to run

1. Requires Oracle 19c / Oracle LiveSQL / Oracle XE.
2. Run `healthcare_inventory_system_final.sql` top-to-bottom as a single script —
   it starts with a cleanup section (drops existing objects if present) so it's
   safe to re-run.
3. Sample data is inserted as part of the script; the required queries at the end
   can be run directly against it.

To view the UI prototype, simply open `healthcare_inventory_redesigned.html` in a
browser — no server or database connection needed, since it runs entirely on
hardcoded sample data.

---

## Status

The SQL backend is complete and functional. The HTML file is a visual prototype
only and is not wired up to the database — connecting the two (e.g., via Oracle
APEX or a lightweight backend) is a potential next step.
