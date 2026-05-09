# Acceptance Criteria

Acceptance criteria define the testable conditions that confirm each requirement is met.

## BR-01 — Onboarding Case Initiation

- A New Hire Form is available in the app to authorised HR Business Partners.
- On submission, a unique Onboarding Case is created with status Not Started.
- The case is timestamped with the date and time of creation.
- The case is automatically assigned to the submitting HRBP.
- A confirmation message is displayed showing the case ID and new hire name.

## BR-02 — New Hire Form Validation

- The form does not submit if any required field is blank: Full Name, Job Role, Department, Start Date, Reporting Manager, Employment Type, Work Location, Personal Email.
- Start Date must be a future date; past dates are rejected with a clear error.
- Personal Email must be in a valid format; invalid entries are rejected.
- Department and Reporting Manager must be selected from existing dropdowns.
- Validation errors display next to the offending field.

## BR-03 — Automatic Task Generation

- On case creation, all standard onboarding tasks for the new hire's role are auto-generated.
- Each task has a due date calculated relative to the start date.
- Role-specific tasks generate only for matching roles; universal tasks generate for all new hires.
- Each task references its source template for traceability.
- Each task defaults to Not Started.

## BR-04 — Automatic Task Assignment

- IT-related tasks are assigned to the IT Administrator.
- Facilities tasks are assigned to the Facilities Officer.
- Compliance tasks are assigned to the Compliance Officer.
- HR tasks are assigned to the HR Business Partner.
- If the primary assignee is on leave, the task falls back to a designated backup.
- The assignee's name is visible on the task.

## BR-05 — Notifications to Task Owners

- The assignee receives an email and Microsoft Teams notification within 5 minutes of task creation.
- The notification includes task name, new hire name, start date, due date, and a direct link.
- The assignee can mark the task complete from the Teams notification.
- A reminder is sent 24 hours before the due date if the task is not yet complete.

## BR-06 — Task Status Updates

- Assignees can update task status from Not Started to In Progress to Completed.
- On completion, the system records the completion date and the user who completed it.
- Comments can be added at any time.
- Once completed, only the HRBP or Head of HR can reopen a task.

## BR-07 — Equipment Allocation Tracking

- Completing an equipment-related task requires entering equipment type and serial number.
- The equipment record is automatically linked to the new hire and the originating task.
- One onboarding case can have multiple equipment records.
- Serial numbers must be unique across the system.
- Equipment is visible on both the new hire's profile and the case detail view.

## BR-08 — Overdue Task Reminders and Escalation

- A reminder email is sent to the assignee on the due date if the task is not complete.
- One day after the due date, the assignee and HRBP are emailed.
- Three days after the due date, the Head of HR is emailed.
- Reminders stop immediately when the task is marked complete.
- Overdue tasks display with a clear visual indicator on the dashboard.

## BR-09 — Day-1 Readiness Check

- A readiness check runs three days before the start date.
- Critical tasks are evaluated; if any are incomplete, the case is flagged At Risk and HR leadership is emailed the outstanding list.
- If all critical tasks are complete, the case is flagged Ready.
- A final readiness check runs the day before the start date.
- The readiness status is visible on the case and the dashboard.

## BR-10 — Onboarding Dashboard

- The dashboard shows total active cases, cases starting in the next 7 days, overdue task count, and Day-1 Readiness summary.
- HR Business Partners see only their assigned cases by default.
- Head of HR sees all cases.
- Filters include Department, Start Date range, HRBP, and Case Status.
- Clicking a case opens the full case detail.

## BR-11 — Automatic Case Closure

- A case auto-closes when all its tasks are marked Completed and the probation outcome is recorded.
- The case closed date is recorded automatically.
- A closure notification is sent to the HRBP, the reporting manager, and the new hire.
- Closed cases are read-only except for the Head of HR.

## BR-12 — Audit Trail

- Every change records the user, timestamp, field changed, old value, and new value.
- The audit log is viewable on each case detail page.
- The audit log cannot be edited or deleted.
- Audit data is retained for at least 2 years.

## BR-13 — Reporting

- Standard reports include average time to complete onboarding, overdue tasks by department, Day-1 Readiness rate, and onboarding volume by month.
- Reports are exportable to Excel and PDF.
- Reports support date range filters.
- Reports can be scheduled for email delivery.

## BR-14 — New Hire Self-Service Access

- The new hire receives a secure portal link after the case is created.
- The new hire can view their start date, manager, and Day-1 logistics.
- The new hire can upload required personal documents.
- The HRBP is notified when documents are uploaded.
- The new hire cannot see internal staff tasks.

## BR-15 — Probation Milestone Tracking

- 30/60/90-day milestone tasks are auto-created on case creation.
- Each milestone is assigned to the reporting manager.
- The manager is reminded three days before each milestone due date.
- The 90-day milestone requires a recorded outcome: Passed, Extended, or Failed.
- The case cannot be fully closed until the 90-day outcome is recorded.

## NFR-01 — Security and Access Control

- HRBPs can only access cases assigned to them.
- Operational staff can only access tasks assigned to them.
- Head of HR has full access.
- New hires can only access their own case data.

## NFR-02 — Performance

- Form submission and case creation complete within 5 seconds.
- Dashboards load within 3 seconds.

## NFR-03 — Availability

- The system is available 99% of business hours.

## NFR-04 — Data Privacy

- Personal data is stored encrypted.
- Personal data is accessible only to authorised HR roles.

## NFR-05 — Mobile Access

- The system is usable on mobile devices for HR Business Partners and managers.

## NFR-06 — Audit Retention

- Audit data is retained for at least 2 years.

## NFR-07 — Notification Delivery

- Notifications reach assigned staff within 5 minutes of task creation.
