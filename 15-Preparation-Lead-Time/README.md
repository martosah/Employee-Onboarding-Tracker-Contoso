# 15. Preparation Lead Time

Most onboarding failures trace back to a single root cause: HR simply didn't have enough time between learning about a hire and that hire walking through the door. **Preparation Lead Time** measures exactly that, turns it into a number on every record, and uses it to catch trouble at the earliest possible moment.

This metric was added in Phase 2 as a deliberate enhancement beyond the original specification.

---

## What the Metric Measures

Preparation Lead Time is the number of days between when an Onboarding Record is opened and the new hire's start date:

> **Preparation Lead Time (Days) = Start Date − Record Opened Date**

A value of 14 means HR had two weeks to prepare. A value of 2 means almost no runway. Negative values are deliberately allowed (the column accepts a range from −365 to 365): a negative lead time means the record was opened *after* the start date — itself a finding worth surfacing, not hiding.

---

## Design Decision — Stored, Not Calculated

The metric could have been a formula (calculated) column that computes live, or a stored whole-number column written by a flow. The build uses the **stored** approach, and the reason is its downstream use.

A formula column that references a related table (Start Date lives on Employee) has Dataverse limitations: sorting is disabled and chart aggregation is restricted. That would defeat the metric's main purpose — feeding dashboards and triage views. A stored whole-number column is fully **sortable, filterable, and aggregatable**, so it can drive "average lead time by department," "records with lead time ≤ 7," and so on.

The one trade-off of storing a value is staleness if the start date later changes. That risk is closed by a dedicated recalculation flow (below), so the stored approach keeps all its analytical upside with none of the drift.

---

## How It's Calculated

The value is stamped by the **Generate Onboarding Tasks** flow at record creation, immediately after the Employee is read and before the task loop runs. The calculation handles three things deliberately:

- It strips the time portion from both dates (`formatDateTime(..., 'yyyy-MM-dd')`) so a record opened late in the day doesn't miscount by one.
- It falls back to the record's creation timestamp if the opened date is somehow blank (`coalesce`).
- It converts the tick difference to whole days by dividing by ticks-per-day, returning a clean integer.

The field is **read-only on the form** — it is flow-maintained, and users must not overwrite it.

---

## Keeping It Accurate

If HR edits a hire's start date after the record exists, the stored lead time would go stale. The **Recalculate Preparation Lead Time** flow (Section 12) prevents this: it triggers when an Employee's start date changes, finds the related Onboarding Record, and re-stamps the value using the same calculation.

One deliberate boundary: recalculation re-stamps the **number** but does **not** re-run the at-intake risk evaluation. The automated risk flag is an *intake* signal — once a record is live, readiness is the HRBP's judgement call, and silently re-flagging on a date edit could overwrite a human decision. Lead time stays current; readiness stays under human control.

---

## At-Intake Risk Flagging

The metric's first and most valuable use is an **early-warning signal**. The Generate Onboarding Tasks flow reads the *Day 1 Readiness Lead Days* environment variable (the minimum acceptable notice) and, the moment a record is created, compares the new record's lead time against it. If the lead time is below the threshold, the flow sets **Day 1 Readiness to At Risk** and emails the HRBP immediately.

This is the difference between a monitoring system and an early-warning system. The scheduled readiness check already catches gaps a few days before the start date; at-intake flagging catches a compressed timeline **the instant the record is opened**, when there's the most time to react.

The comparison uses "is less than," which encodes the rule correctly: meeting the minimum notice is compliant; only falling *short* of it flags. This was verified in testing — records opened with 1 and 2 days' lead were flagged At Risk; a record opened with exactly the 3-day minimum stayed Pending; records with 7 and 14 days stayed Pending.

---

## How the Metric Is Used Beyond the Form

The stored, aggregatable design unlocks several uses that turn a task list into a process-health instrument. One is built; the others are documented as the metric's intended operational value:

1. **At-intake risk flagging** *(built)* — the early-warning behaviour above.
2. **Born-overdue task detection** — templates carry negative due-date offsets (e.g. T-10). If a hire's lead time is shorter than an offset, that task is created already overdue. The metric lets the system recognise a compressed timeline and avoid treating unavoidable lateness as a genuine escalation.
3. **Dashboard KPIs** — average Preparation Lead Time by department, by HRBP, and trended monthly answers a question leadership actually asks: *is recruiting giving HR enough notice, and is it improving?*
4. **Triage view** — a "Compressed Onboarding" view filters records with lead time ≤ 7 and sorts ascending, so HRBPs work the tightest timelines first each morning. (Views can't read environment variables, so this uses a static threshold.)
5. **Root-cause evidence** — over time, correlating lead time against readiness and probation outcomes can produce a data-backed policy recommendation (for example, "records must be opened at least 10 working days before start"). That is the kind of evidence a steering committee acts on.

---

## Historical Records

Records created before this column existed (the earliest eighteen) have a blank Preparation Lead Time. This is handled honestly rather than back-filled blindly:

- Charts and averages ignore nulls, so dashboards remain correct — they simply reflect records from the metric's introduction onward.
- The metric is documented as **effective from its introduction date**, so the gap is explicit.
- A one-time back-fill (computing the value for historical records from their existing dates) is noted as a future option rather than run retroactively.

---

➡️ Next: **[Section 16 — Solution Hygiene and ALM](../16-Solution-Hygiene-and-ALM)**
