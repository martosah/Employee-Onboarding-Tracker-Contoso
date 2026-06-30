# 12. Automation and Flows

The data model holds the information; the flows make the system *behave*. The solution has **22 Power Automate cloud flows in total** — **17 backend flows**, documented here, that turn a new Onboarding Record into a fully generated, assigned, notified, tracked, and self-closing onboarding case, and **5 portal flows**, added in Phase 2 Part 2, that drive the Power Pages document lifecycle (documented in [Section 18](../18-Document-Lifecycle-and-Portal-Automation)).

This section groups the backend flows by what they do, then records the engineering decisions that made them work reliably. Several of those decisions came from real failures during the build and are documented honestly, because they are the kind of platform behaviour that isn't obvious until you hit it.

---

## How the Flows Fit Together

The 17 backend flows fall into seven functional groups:

| Group | Flows | What it does |
| --- | :---: | --- |
| Record & Task Generation | 2 | Creates the record and generates its tasks from templates |
| Notifications | 3 | Event-driven emails to the right person at the right moment |
| Scheduled Checks | 3 | Time-driven reminders, escalations, and readiness evaluation |
| Status Automation | 4 | Keeps record status, completion metadata, and readiness in sync |
| Process Flow Sync | 1 | Advances the business process flow bar |
| Equipment & Documents | 2 | Handles equipment provisioning and document naming |
| Metric Maintenance | 2 | Maintains the Preparation Lead Time metric |

A further five flows form an eighth group — **Portal Document Lifecycle** — covered separately in Section 18, because they belong to the Power Pages build rather than the internal solution.

---

## Flow Inventory

| # | Flow | Trigger | Purpose |
| --- | --- | --- | --- |
| 1 | Auto-Create Onboarding Record | Employee created | Creates the Onboarding Record for a new hire automatically |
| 2 | Generate Onboarding Tasks | Onboarding Record created | Generates tasks from matching templates, assigns owners, sets the HRBP owner, stamps lead time |
| 3 | Notify Assignee – Task Created | Onboarding Task created | Emails the task owner |
| 4 | Notify HRBP – Document Uploaded | New Hire Document created | Emails the record's HRBP to review the document |
| 5 | Welcome New Hire – Record Created | Onboarding Record created | Sends the welcome email with the portal invite link |
| 6 | Task Due-Date Reminder | Scheduled (daily) | Reminds owners of tasks due tomorrow |
| 7 | Overdue Task Escalation | Scheduled (daily) | Chases overdue tasks, copies Head of HR |
| 8 | Day-1 Readiness Check | Scheduled (daily) | Three days before start, flags records with incomplete critical tasks |
| 9 | Day 1 Readiness | Scheduled / final check | Final readiness evaluation closer to the start date |
| 10 | Set Record In Progress | Onboarding Task modified | Moves a record from Not Started to In Progress on first activity |
| 11 | Stamp Task Completion | Onboarding Task modified | Sets Completed Date and Completed By when a task completes |
| 12 | Set Readiness Ready on Critical Completion | Onboarding Task modified | Sets Day-1 Readiness to Ready when critical tasks are done |
| 13 | Auto-Close Record | Onboarding Task modified | Closes the record when all tasks complete and probation is recorded |
| 14 | Sync BPF Stage to Status | Onboarding Record modified | Advances the business process flow stage |
| 15 | Equipment Provisioning Prompt | Onboarding Task created | Handles equipment-related tasks |
| 16 | Document Title | New Hire Document created | Names the document (hire name + document type) |
| 17 | Recalculate Preparation Lead Time | Employee modified (start date) | Re-stamps lead time when a start date changes |

> Flow names carry no number prefix. During the portal build the naming was standardised so every flow is named purely by what it does (the descriptive name *is* the identifier), which keeps the inventory self-documenting and avoids implying an execution order that event-triggered flows don't have.

---

## The Portal Flows (Detailed in Section 18)

Five flows added in Phase 2 Part 2 drive the Power Pages document lifecycle. They are listed here for inventory completeness; the full build, triggers, and portal-specific platform behaviours are in [Section 18](../18-Document-Lifecycle-and-Portal-Automation).

| Flow | Trigger | Purpose |
| --- | --- | --- |
| Provision New Hire Portal Access | Onboarding Record created/updated | Creates or links the portal Contact and binds it to the record |
| Reset Document Status on Re-upload | New Hire Document file added or modified | Flips a rejected document back to Pending Review and clears the review note on resubmission |
| Archive Rejected Document & Free Slot | New Hire Document status → Rejected | Archives the rejected file to the Rejected Document Archive table and frees the upload slot |
| Notify New Hire on Review Outcome | New Hire Document status modified | Emails the hire the approve/reject outcome with the review note |
| Duplicate Document Catcher | New Hire Document created | Auto-rejects a new document if an active one of the same type already exists for that contact |

---

## The Generation Spine

Two flows do the heavy lifting, and the rest react to what they create.

**Auto-Create Onboarding Record** fires when an Employee record is created, so onboarding begins the moment a new hire exists in the system — no one has to remember to open a record.

**Generate Onboarding Tasks** is the most complex flow in the solution. On record creation it:

1. Reads the related Employee (for start date and job role).
2. Lists applicable Task Templates using an OData filter that returns both universal templates and templates matching the hire's specific job role.
3. Stamps the **Preparation Lead Time** and evaluates at-intake risk (covered in Section 15).
4. Sets the record's owner to the assigned HRBP — **before** the task loop runs.
5. Loops the templates, creating one Onboarding Task each: due date computed as start date plus the template's offset (`addDays`), owner set to the resolved staff member, and the new-hire start date stamped onto the task.

The ordering in step 4 is not cosmetic — it is forced by the parental relationship (see "Parental-cascade ordering" below).

---

## Notifications, Scheduled Checks, and Status Automation

The **notification** flows share one skeleton: trigger on a row event, check the *Enable Email Notifications* environment variable as a master switch, resolve the recipient by traversing lookups to the right Staff record, and send through the Outlook connection reference. Welcome and document-review emails follow the same shape.

The **scheduled** flows run on recurrence triggers and query Dataverse by date. The Due-Date Reminder finds tasks due tomorrow; Overdue Escalation finds incomplete tasks past their due date and copies Head of HR; the Readiness checks evaluate records approaching their start date, count incomplete critical tasks, and set Day-1 Readiness to Ready or At Risk — emailing the outstanding task list to the HRBP and leadership when there's a gap.

The **status automation** flows keep the record honest without anyone editing it: first task activity moves the record to In Progress; completing a task stamps who completed it and when; completing the critical tasks flips readiness to Ready; and once every task is complete and a probation outcome is recorded, the record closes itself.

---

## Engineering Decisions

These are the decisions and platform behaviours that shaped the backend flows. Each is recorded because it changed how a flow had to be built. (The portal flows carry their own distinct set of platform learnings — the two-step annotation commit, `@odata.bind` entity-set paths, and loop-guard triggers — documented in Section 18.)

**Reading user identity from Staff, not the system-user metadata endpoint.** The original design resolved a task owner by looking up the system user via the connector's metadata endpoint. This failed repeatedly and unrecoverably with a metadata/deserialisation error — confirmed as an intermittent platform-side issue with the systemuser table, not a design flaw. The robust fix was to add an **App User** lookup column on the Staff table, populate each staff member's system-user identity once, and have the flow read the GUID directly from the Staff record it already fetches. Ownership is then set with a bind expression of the form `concat('systemusers(', <App User value>, ')')`. This eliminated the per-run metadata call entirely.

**Parental-cascade ordering.** Because Onboarding Record → Onboarding Task is a parental relationship, setting the parent record's owner cascades that owner down to its child tasks. In the first build, the record owner was set *after* the task-creation loop, which overwrote every task's individual owner with the HRBP. The fix was to set the record owner **before** the loop, so per-task ownership survives.

**Choice fields: numeric in conditions, label in updates.** Choice columns store an integer (prefixed `89107`, e.g. Not Started = `891070000`, In Progress = `891070001`). Two rules emerged: comparisons in a Condition must use the raw integer, because the human-readable FormattedValue annotation is **not available on trigger outputs**; but an Update a row action sets the choice using its dropdown **label**. Mixing these up was the cause of several early failures.

**Yes/No environment variables store the lowercase string "yes".** The Enable Email Notifications guard initially failed because the condition compared against `Yes` or `true`. A Yes/No environment variable stores its value as the lowercase string `yes` — the comparison had to match exactly.

**Standard environment-variable read pattern.** Every flow reads a variable the same way: a List rows on the *Environment Variable Definitions* table filtered by `schemaname` (lowercase), with the value extracted via `first(outputs(...))?['body/value'])?['defaultvalue']`. This is consistent across the readiness threshold, the portal URL, the notification inboxes, and the master switch.

**Resolving lookup display names without extra calls.** To show a template or record name rather than a GUID, flows use the FormattedValue OData annotation directly on the lookup value (`...['_pex_tasktemplate_value@OData.Community.Display.V1.FormattedValue']`), avoiding an additional Get row action per loop.

**Avoiding the auto-generated Apply to each.** Selecting a list-sourced value from the dynamic-content picker silently wraps the action in an unwanted Apply to each loop. Where a flow needs a single value from the trigger, the expression is typed directly in the **Expression tab** (e.g. `triggerOutputs()?['body/_pex_onboardingrecord_value']`), which keeps it a scalar and avoids the spurious loop.

**Advancing the business process flow via its instance table.** Moving the BPF bar by writing the deprecated `stageid` on the record did not work — it rejects both bind syntax and bare GUIDs. The working approach (detailed in Section 13) reads the BPF instance with a Get row, then updates the **business process flow instance table**, setting Active Stage with a bind expression. This is the supported way to drive a BPF from automation.

**Guarding status transitions.** Set Record In Progress originally checked only for the In Progress value and missed records where a task jumped straight to Completed. The condition was broadened to "status is not equal to Not Started," covering both paths, while retaining a guard that only acts on records currently Not Started so it never overwrites a later status.

---

➡️ Next: **[Section 13 — Business Process Flow](../13-Business-Process-Flow)**
