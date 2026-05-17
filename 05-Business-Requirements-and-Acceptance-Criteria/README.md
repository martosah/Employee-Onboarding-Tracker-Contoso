# 5. Business Requirements and Acceptance Criteria

Each business requirement is paired with its acceptance criteria — the testable conditions that confirm the requirement is met. Requirements are grouped by lifecycle stage and prioritised on the Must-have / Should-have scale.

---

## Group 1 — record Initiation

### BR-01 — HR Business Partners can create onboarding record when an offer is signed

**Statement:** HR Business Partners must be able to initiate an onboarding record by submitting a New Hire Form as soon as the new hire's offer letter is signed and returned.

**Acceptance Criteria:**

1. Given a logged-in HR Business Partner, when they open the New Hire Form, they should see all required fields for capturing the new hire's details.
2. Given a completed and submitted form, a unique onboarding record should be created with status Not Started and timestamped automatically.
3. Given a newly created record, the submitting HRBP should be automatically assigned as the record owner.
4. Given a successful submission, a confirmation message should display the record ID and new hire name.

**Priority:** Must-have

---

### BR-02 — New Hire Form must validate all critical information before a record is created

**Statement:** The system must enforce that all critical onboarding information is captured and valid before allowing a record to be created.

**Acceptance Criteria:**

1. Given an incomplete form, when the HRBP attempts to submit, the form must reject the submission and indicate the missing required field.
2. Given a Start Date in the past, the form must reject the submission with a clear error message.
3. Given an invalid Personal Email format, the form must reject the submission with an inline error.
4. Given a free-text entry in Department or Reporting Manager, the form must reject the entry — these must be selected from dropdowns.

**Priority:** Must-have

---

## Group 2 — Task Generation and Assignment

### BR-03 — Standard onboarding tasks must be auto-generated when a record is created

**Statement:** Once an onboarding record is created, the system must automatically generate the standard set of onboarding tasks for the new hire's role, with due dates calculated relative to the Start Date.

**Acceptance Criteria:**

1. Given a newly created record, all universal onboarding tasks must be auto-generated for the new hire.
2. Given a new hire's specific job role, additional role-specific tasks must be auto-generated; tasks for other roles must be excluded.
3. Given each generated task, the due date must be calculated as Start Date plus the template's offset.
4. Given each generated task, it must reference its source template for traceability and default to status Not Started.

**Priority:** Must-have

---

### BR-04 — Tasks must be auto-assigned to the correct staff owner

**Statement:** Each generated task must be automatically assigned to the appropriate staff member based on task type, with a fallback to a designated backup when the primary owner is unavailable.

**Acceptance Criteria:**

1. Given an IT-related task, it must be assigned to the IT Administrator.
2. Given a Facilities task, it must be assigned to the Facilities Officer.
3. Given a Compliance task, it must be assigned to the Compliance Officer.
4. Given an HR task, it must be assigned to the HR Business Partner who owns the record.
5. Given the primary assignee is on leave, the task must automatically fall back to the designated Backup Staff.

**Priority:** Must-have

---

## Group 3 — Notifications, Reminders and Escalations

### BR-05 — Task owners must be notified when tasks are assigned

**Statement:** The system must notify the relevant staff member whenever a task is assigned to them or its due date approaches.

**Acceptance Criteria:**

1. Given a new task is created, the assignee must receive both an email and Microsoft Teams notification within 5 minutes.
2. Given a notification, it must contain the task name, new hire name, Start Date, Due Date, and a direct link to the task.
3. Given a Teams notification, the assignee must be able to mark the task complete directly from the Adaptive Card.
4. Given a task is not yet completed 24 hours before its due date, a reminder email must be sent to the assignee.

**Priority:** Must-have

---

### BR-06 — Overdue tasks must be reminded and escalated through the HR hierarchy

**Statement:** The system must automatically chase overdue tasks and escalate them through HR leadership if they remain incomplete.

**Acceptance Criteria:**

1. Given a task is overdue on its due date, a reminder email must be sent to the assignee.
2. Given a task is one day overdue, the assignee must receive a second reminder with the HRBP copied.
3. Given a task is three days overdue, the Head of HR must be emailed with full record context.
4. Given a task is marked complete at any point, all reminder and escalation chains must stop immediately.
5. Given an overdue task, it must display with a clear visual indicator (red flag) on the dashboard.

**Priority:** Must-have

---

## Group 4 — Task Execution and Completion

### BR-07 — Assignees must be able to update task status and add notes

**Statement:** The system must allow assigned staff to update task status, add notes, and complete tasks.

**Acceptance Criteria:**

1. Given an assigned task, the assignee must be able to change status from Not Started to In Progress to Completed.
2. Given a completed task, the system must record the completion date and the user who completed it.
3. Given any task, comments must be addable at any stage.
4. Given a completed task, only the HRBP or Head of HR may reopen it.

**Priority:** Must-have

---

### BR-08 — Equipment must be tracked when issued during onboarding

**Statement:** The system must record all equipment issued to a new hire and link it to their onboarding record and employee profile.

**Acceptance Criteria:**

1. Given an equipment-related task being marked complete, the assignee must be required to enter equipment type and serial number before save is allowed.
2. Given a completed equipment task, an equipment record must be created and automatically linked to the new hire and the originating task.
3. Given a new hire, multiple equipment records (laptop, headset, monitor, etc.) may be associated with their onboarding record.
4. Given an equipment record, the serial number must be unique across the entire system.
5. Given an equipment record, it must be visible on both the new hire's profile and the record detail view.

**Priority:** Must-have

---

## Group 5 — Day-1 Readiness and record Visibility

### BR-09 — The system must confirm Day-1 readiness before the new hire's start date

**Statement:** Before a new hire's Start Date, the system must confirm that all critical tasks are complete and flag any gaps to HR leadership.

**Acceptance Criteria:**

1. Given three days before a new hire's Start Date, the system must run a readiness check on all critical tasks.
2. Given any incomplete critical tasks, the record must be flagged At Risk and HR leadership (HRBP and Head of HR) emailed the outstanding list.
3. Given all critical tasks are complete, the record must be flagged Ready.
4. Given the day before the Start Date, a final readiness check must run and escalate any remaining gaps to the Reporting Manager.
5. Given any readiness status, it must be visible on both the record detail view and the dashboard.

**Priority:** Must-have

---

### BR-10 — HR must have a single dashboard view of all onboarding activity

**Statement:** HR Business Partners and HR leadership must have a single dashboard view showing all onboarding activity in real time.

**Acceptance Criteria:**

1. Given a logged-in HRBP, the dashboard must show only records assigned to them by default, with a toggle to view all if needed.
2. Given a logged-in Head of HR, the dashboard must show all records organisation-wide.
3. Given the dashboard, it must display total active records, records starting in the next 7 days, overdue task count, and Day-1 Readiness summary.
4. Given dashboard filters, users must be able to filter by Department, Start Date range, HRBP, and record Status.
5. Given a record row on the dashboard, clicking it must open the full record detail view.

**Priority:** Must-have

---

## Group 6 — record Closure and Audit

### BR-11 — records must auto-close when all tasks and probation requirements are met

**Statement:** The system must mark an onboarding record as Completed only when all its tasks are completed and the probation outcome is recorded.

**Acceptance Criteria:**

1. Given all tasks on a record are marked Completed and the Probation Outcome is recorded, the record must auto-close.
2. Given an auto-closed record, the record Closed Date must be recorded automatically.
3. Given a closed record, a closure notification must be sent to the HRBP, Reporting Manager, and the new hire.
4. Given a closed record, it must become read-only for everyone except the Head of HR.

**Priority:** Must-have

---

### BR-12 — Every change must be captured in a tamper-proof audit trail

**Statement:** The system must maintain a complete, tamper-proof history of all changes made to records and tasks.

**Acceptance Criteria:**

1. Given any change to a record, task, employee, equipment, or document record, the system must log the user, timestamp, field changed, old value, and new value.
2. Given a record detail view, the audit log must be viewable in a dedicated tab.
3. Given an audit log entry, it must not be editable or deletable by any user, including administrators.
4. Given audit data, it must be retained for at least two years.

**Priority:** Must-have

---

## Group 7 — Reporting

### BR-13 — HR leadership must be able to generate reports on onboarding performance

**Statement:** HR leadership must be able to generate reports on onboarding performance for trend analysis and process improvement.

**Acceptance Criteria:**

1. Given standard reports, they must include average time to complete onboarding, overdue tasks by department, Day-1 Readiness rate, and onboarding volume by month.
2. Given any report, it must be exportable to Excel and PDF.
3. Given any report, it must support date range filters and grouping by Department or HRBP.
4. Given the Head of HR, they must be able to schedule reports for automatic monthly email delivery.

**Priority:** Should-have

---

## Group 8 — New Hire Self-Service

### BR-14 — New hires must be able to view their own onboarding and upload personal documents

**Statement:** The new hire must be able to access a secure portal to view their onboarding progress, see Day-1 logistics, and upload required personal documents before their Start Date.

**Acceptance Criteria:**

1. Given a newly created record, the new hire must receive a welcome email with a secure portal link.
2. Given a logged-in new hire, they must be able to view their Start Date, Reporting Manager, and Day-1 logistics.
3. Given a logged-in new hire, they must be able to upload required documents (BVN, NIN, academic certificate, bank details).
4. Given a document upload, the HRBP must receive an immediate notification to review and approve.
5. Given a logged-in new hire, they must NOT be able to see internal staff tasks or other new hires' data.

**Priority:** Should-have

---

## Group 9 — Probation Milestones

### BR-15 — The system must track 30/60/90-day probation milestones after the Start Date

**Statement:** The system must track 30/60/90-day milestones following the new hire's Start Date and capture the final probation outcome before the record can be fully closed.

**Acceptance Criteria:**

1. Given a newly created record, three probation milestone tasks must be auto-created for 30, 60, and 90 days after Start Date.
2. Given each milestone task, it must be assigned to the new hire's Reporting Manager.
3. Given a milestone due date, the Reporting Manager must be reminded three days in advance.
4. Given the 90-day milestone task, it cannot be marked complete unless a Probation Outcome (Passed, Extended, or Failed) is selected.
5. Given a record with no recorded Probation Outcome, it cannot auto-close even if all tasks are otherwise complete.

**Priority:** Must-have

---

## Group 10 — Non-Functional Requirements

### NFR-01 — Security and access control must be role-based

**Statement:** Access to data and actions must be enforced by role-based permissions.

**Acceptance Criteria:**

1. Given an HRBP, they must only access records assigned to them.
2. Given operational staff (IT, Facilities, Compliance), they must only access tasks assigned to them.
3. Given the Head of HR, they must have full access across all records.
4. Given a new hire on the portal, they must only access their own record data.

**Priority:** Must-have

---

### NFR-02 — System performance must meet response time targets

**Statement:** The system must meet defined performance targets under normal operating load.

**Acceptance Criteria:**

1. Given a form submission, record creation must complete within 5 seconds.
2. Given a dashboard request, it must load within 3 seconds.

**Priority:** Must-have

---

### NFR-03 — The system must be available during business hours

**Statement:** The system must be available 99% of business hours.

**Acceptance Criteria:**

1. Given business hours (8am to 6pm, Monday to Friday), the system must be available 99% of the time.

**Priority:** Must-have

---

### NFR-04 — Personal data must be stored securely

**Statement:** Personal data must be encrypted at rest and accessible only to authorised HR roles.

**Acceptance Criteria:**

1. Given personal data fields (BVN, NIN, bank details), they must be stored encrypted in the database.
2. Given personal data, access must be restricted to authorised HR roles only.

**Priority:** Must-have

---

### NFR-05 — The system must support mobile access

**Statement:** HR Business Partners and managers must be able to use the system on a mobile device.

**Acceptance Criteria:**

1. Given an HRBP or Reporting Manager on a mobile device, they must be able to view records, update tasks, and approve documents.

**Priority:** Should-have

---

### NFR-06 — Audit data must be retained for at least two years

**Statement:** Audit log data must be retained for a minimum of two years to support compliance and investigation needs.

**Acceptance Criteria:**

1. Given audit data, it must be retained for at least 24 months.

**Priority:** Must-have

---

### NFR-07 — Notifications must be delivered within 5 minutes

**Statement:** Notifications must reach assigned staff within 5 minutes of task creation or status change.

**Acceptance Criteria:**

1. Given a newly created or reassigned task, the notification must reach the assignee within 5 minutes.

**Priority:** Must-have

---

➡️ Next: **[Section 6 — Entity Relationship Diagram](../06-Entity-Relationship-Diagram)**
