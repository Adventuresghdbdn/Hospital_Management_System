# Hospital Management System — SQL Server Data Analyst Portfolio Project

A self-contained hospital analytics database built in SQL Server Management Studio (SSMS).
Designed for a data-analyst interview: the schema is deliberately lean (8 tables) so the
bulk of the project is the analysis layer — joins, CTEs, window functions, views, and a
reusable reporting layer — not database plumbing.

## How to run it

Open each file in SSMS and execute in this order:

1. **`01_schema.sql`** — creates `HospitalManagementDB` and all 8 tables with PK/FK/CHECK constraints.
2. **`02_seed_data.sql`** — generates realistic sample data: 10 departments, 25 diagnoses,
   60 rooms, 50 doctors, 800 patients, 3,000 appointments, 500 admissions, ~2,300 billing rows.
3. **`03_analytical_queries.sql`** — 10 business-question queries. This is what to walk through in the interview.
4. **`04_views_procedures_triggers.sql`** — 2 views, 2 parameterized stored procedures, 1 index, 2 triggers.

Each script is idempotent (drops/recreates objects), so you can re-run the whole thing from scratch at any time.

## Entity relationship overview

```
Departments 1---* Doctors
Departments 1---* Rooms
Patients    1---* Appointments  *---1 Doctors
Patients    1---* Admissions    *---1 Doctors
Admissions  *---1 Rooms
Admissions  *---1 Diagnoses
Billing     *---1 Patients   (and optionally *---1 Admissions or *---1 Appointments)
```

Draw this out in SSMS's **Database Diagrams** feature (right-click Database Diagrams →
New Database Diagram → add all 8 tables) — SSMS will auto-lay-out the FK relationships,
and having that diagram ready to screen-share is worth more in the interview than
describing it verbally.

## The 8 tables

| Table | Purpose |
|---|---|
| Departments | Hospital departments (Cardiology, Emergency, etc.) |
| Doctors | Doctor roster, linked to a department |
| Patients | Patient demographics |
| Rooms | Beds/wards, linked to a department, status auto-managed by triggers |
| Diagnoses | Reference list of diagnoses with category |
| Appointments | Outpatient visits — Scheduled/Completed/NoShow/Cancelled |
| Admissions | Inpatient stays — links patient, doctor, room, diagnosis |
| Billing | Revenue events, tied to either an appointment or an admission |

## Business questions answered (see `03_analytical_queries.sql`)

1. Monthly revenue by department, with month-over-month change
2. Average length of stay by diagnosis
3. Top doctors by patient volume and their no-show rate
4. Bed occupancy rate by ward type
5. Age group / gender breakdown of diagnoses
6. 30-day readmission rate
7. Peak admission day of week (staffing insight)
8. Revenue per patient, ranked by doctor
9. Monthly no-show trend
10. Running total of monthly revenue

## Design decisions worth explaining in the interview

- **Normalization**: schema is in 3NF — no repeated groups, diagnoses and departments are
  reference tables rather than free-text fields, which is what makes the GROUP BY analysis clean.
- **Billing dual-source design**: a bill can originate from either an appointment or an
  admission (`CHK_Billing_Source` enforces at least one is present) rather than forcing
  two separate billing tables — a realistic trade-off you can defend.
- **Trigger-driven room status**: `Rooms.Status` updates automatically on admit/discharge
  instead of being manually maintained, which avoids a common data-integrity bug (stale
  "Occupied" rooms) and shows you can enforce business rules in the database.
- **Covering index**: `IX_Appointments_DoctorID_Status` was added because Q3 and
  `vw_DoctorPerformance` both filter/group by `(DoctorID, Status)` and select
  `AppointmentDate` — the `INCLUDE` avoids a key lookup. Pull up the execution plan
  before/after in SSMS if asked about performance.
- **Synthetic data caveats**: dates, names, and amounts are randomly generated
  (`NEWID()`/`CHECKSUM`), so a few edge cases (e.g. a diagnosis with only 1-2 cases)
  will look noisy — worth mentioning proactively, since real analyst work involves
  flagging exactly this kind of small-sample noise.

## Next steps to make it interview-ready

- Connect SSMS's DB to **Power BI or Excel** and build one dashboard (e.g. revenue by
  department + no-show trend) — this is the "insight communication" layer interviewers
  for data-analyst roles specifically look for.
- Push this folder to GitHub with the ER diagram screenshot and a couple of dashboard
  screenshots in the README.
- Rehearse narrating 2-3 of the queries out loud: what question it answers, why you
  chose that SQL construct (e.g. why `LAG()` over a self-join for Q6), and what the
  result actually means for hospital operations.
