
# Business Logic

The rules below define how the system behaves automatically. Each rule has a trigger, a condition, and an action.

## Form Validation

**BL-01 — Required Fields**
On New Hire Form submit, block submission if any required field is blank.

**BL-02 — Future Start Date**
Reject submission if Start Date is not in the future.

**BL-03 — Email Format**
Reject submission if Personal Email is not in valid format.

**BL-04 — Unique Employee Number**
The Employee Number Alternate Key prevents duplicate employee numbers.

## Case and Task Generation

**BL-05 — Case Auto-Creation**
When an Employee record is inserted, create one Onboarding Case with status Not Started, the current timestamp, and the submitting user as Assigned HRBP.

**BL-06 — Task Generation from Templates**
When a case is created, generate one Onboarding Task per matching Task Template (templates that apply to all roles, plus templates matching the new hire's job role). Each task's due date is calculated as Start Date plus the template's offset.

**BL-07 — Assignee Resolution**
For each generated task, resolve the assignee by Staff Role. If the primary assignee is on leave, fall back to their Backup Staff. If no backup is available, escalate to the role's department head.

**BL-08 — Probation Tasks**
On case creation, create three additional tasks for 30, 60, and 90 days after Start Date, assigned to the Reporting Manager.

## Notifications and Reminders

**BL-09 — Task Assignment Notification**
On task creation, send an email and Teams notification to the assignee within 5 minutes containing task name, new hire name, start date, due date, and a direct link.

**BL-10 — Pre-Due Reminder**
24 hours before a task's due date, send a reminder email to the assignee if the task is not Completed.

**BL-11 — Day-Of Reminder**
At 9:00 AM on the due date, send a reminder email to the assignee if the task is not Completed.

**BL-12 — Day-After Escalation**
One day after the due date, email the assignee with the HRBP copied if the task is not Completed.

**BL-13 — Three-Day Escalation**
Three days after the due date, email the Head of HR if the task is not Completed.

**BL-14 — Document Upload Notification**
When a new hire uploads a document, notify the Assigned HRBP.

## Day-1 Readiness

**BL-15 — Three-Day Readiness Check**
At 8:00 AM each day, evaluate every case where Start Date is three days away. If all critical tasks are Completed, set Day-1 Readiness to Ready. Otherwise, set to At Risk and email the HRBP and Head of HR with the list of outstanding critical tasks.

**BL-16 — One-Day Final Check**
At 4:00 PM the day before Start Date, repeat the readiness check. If still At Risk, escalate to the HRBP, Head of HR, and Reporting Manager.

## Task Completion

**BL-17 — Equipment Required for Completion**
A task linked to a template marked Requires Equipment cannot be saved as Completed without entered equipment type and serial number. On save, an Equipment record is created and linked to the new hire and the task.

**BL-18 — Completion Metadata**
When a task's status changes to Completed, set Completed Date to the current timestamp and Completed By to the current user.

**BL-19 — Restricted Reopening**
A user cannot change a task's status from Completed to anything else unless they are the Assigned HRBP or the Head of HR.

## Case Closure

**BL-20 — Auto-Closure**
A case is set to Completed when 100% of its tasks are Completed and a Probation Outcome is recorded. Set Case Closed Date and notify the HRBP, Reporting Manager, and new hire. Make the case read-only except for Head of HR.

**BL-21 — Probation Outcome Required**
The 90-day probation task cannot be saved as Completed unless a Probation Outcome (Passed, Extended, or Failed) is selected.

## Documents

**BL-22 — Document Title Calculation**
On document creation, set Document Title to "[Employee Full Name] — [Document Type]".

**BL-23 — Rejection Loop**
When a document is set to Rejected, email the new hire with the rejection reason and require re-upload.

## Calculated Fields

**BL-24 — Days Until Start Date**
On the Employee record: Start Date minus Today.

**BL-25 — Case Completion Percentage**
On the Onboarding Case: completed task count divided by total task count, multiplied by 100.

**BL-26 — Days Overdue**
On the Onboarding Task: zero if Completed, otherwise the greater of zero and Today minus Due Date.

## Access Control and Audit

**BL-27 — Row-Level Access**
HRBPs see and edit only cases where they are the Assigned HRBP. Operational staff see and edit only tasks assigned to them. Reporting Managers see their direct reports' cases as read-only. Head of HR sees everything.

**BL-28 — Self-Service Scope**
A new hire sees only their own case as read-only. They can write only to their own document records that are still in Pending Review.

**BL-29 — Audit Logging**
Every change to Onboarding Case, Onboarding Task, Employee, Equipment, and New Hire Document is logged with user, timestamp, field changed, old value, and new value. Audit logs are immutable and retained for two years.
