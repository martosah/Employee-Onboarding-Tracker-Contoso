
# Business Requirements

The system must meet the following functional and non-functional requirements.

## Functional Requirements

**BR-01 — Onboarding Case Initiation**
The system must allow an HR Business Partner to create an onboarding case as soon as a new hire's offer is signed.

**BR-02 — New Hire Form Validation**
The system must enforce that all critical onboarding information is captured and valid before a case is created.

**BR-03 — Automatic Task Generation**
Once a case is created, the system must automatically generate the standard set of onboarding tasks for the new hire's role, with due dates calculated relative to the start date.

**BR-04 — Automatic Task Assignment**
Each generated task must be auto-assigned to the correct staff owner based on task type, with fallback to a designated backup if the primary owner is on leave.

**BR-05 — Notifications to Task Owners**
The system must notify the relevant staff member whenever a task is assigned, reassigned, or its due date approaches.

**BR-06 — Task Status Updates**
The system must allow assigned staff to update task status, add notes, and complete tasks.

**BR-07 — Equipment Allocation Tracking**
The system must record all equipment issued to a new hire and link it to their onboarding case and employee profile.

**BR-08 — Overdue Task Reminders and Escalation**
The system must automatically chase overdue tasks and escalate them through the HR hierarchy if they remain incomplete.

**BR-09 — Day-1 Readiness Check**
Before a new hire's start date, the system must confirm that all critical tasks are complete and flag any gaps to HR leadership.

**BR-10 — Onboarding Dashboard**
HR Business Partners and HR leadership must have a single dashboard view showing all onboarding activity.

**BR-11 — Automatic Case Closure**
The system must mark an onboarding case as Completed only when all its tasks are completed and the probation outcome is recorded.

**BR-12 — Audit Trail**
The system must maintain a complete, tamper-proof history of all changes made to cases and tasks.

**BR-13 — Reporting**
HR leadership must be able to generate reports on onboarding performance.

**BR-14 — New Hire Self-Service Access**
The new hire must be able to view their onboarding progress and upload required personal documents before Day 1.

**BR-15 — Probation Milestone Tracking**
The system must track 30/60/90-day milestones after the start date and capture the final probation outcome.

## Non-Functional Requirements

**NFR-01 — Security and Access Control**
Access must be role-based. HR Business Partners can only access their assigned cases. Operational staff can only access tasks assigned to them. Head of HR has full access. New hires can only access their own data.

**NFR-02 — Performance**
Form submission and case creation must complete within 5 seconds. Dashboards must load within 3 seconds.

**NFR-03 — Availability**
The system must be available 99% of business hours (8am–6pm, Monday to Friday).

**NFR-04 — Data Privacy**
Personal data (BVN, NIN, bank details) must be stored encrypted and accessible only to authorised HR roles.

**NFR-05 — Mobile Access**
HR Business Partners and managers must be able to use the system on a mobile device.

**NFR-06 — Audit Retention**
Audit data must be retained for at least 2 years.

**NFR-07 — Notification Delivery**
Notifications must reach assigned staff within 5 minutes of task creation.
