# 8. Power Platform Components

The solution uses six Power Platform components, supported by Microsoft 365 services for communication.

> **Build status (as of Phase 2).** Four components are built and live: Dataverse, the model-driven app, Power Automate, and — added in **Phase 2 Part 2** — the **Power Pages** portal. The remaining two specified components, **Power BI** and **Copilot Studio**, are designed here but deferred to **Phase 3**. Each component below notes its status.

---

## Dataverse  ·  ✅ Built

The data foundation. Stores the ten tables, relationships, calculated columns, business rules, security roles, and audit configuration. Implements form-level rules and uniqueness constraints that do not require workflows. *(As built: Section 11.)*

---

## Power Apps — Model-Driven App  ·  ✅ Built

The primary interface for HR and operational staff. Hosts the New Hire Form, record detail forms, task forms, dashboards, and views. Used by HR Business Partners, Head of HR, IT Administrators, Facilities Officers, Compliance Officers, and Reporting Managers. *(As built: Section 10.)*

---

## Power Pages  ·  ✅ Built (Phase 2 Part 2)

The self-service portal for new hires. Hosts secure document upload, record status visibility, and Day-1 logistics. Used by new hires before they have corporate accounts. As built, it also provides HR review with approve/reject, document resubmission for rejected items, automated outcome notifications, per-hire row-level data isolation, and duplicate-upload prevention. *(As built: Sections 17–20.)*

---

## Power Automate  ·  ✅ Built

Implements the workflow automation. Three flow types are used:

- **Automated flows** triggered by Dataverse events: record creation, task generation, task notification, equipment validation, completion metadata, auto-closure, and the portal document lifecycle (upload notification, status reset on re-upload, rejected-document archiving, review-outcome notification, and duplicate detection).
- **Scheduled flows** running daily: pre-due reminders, day-of reminders, escalations, three-day and one-day readiness checks.
- **Manual flows** triggered from forms: record reassignment and ad-hoc readiness re-runs.

*(Backend flows: Section 12. Portal flows: Section 18.)*

---

## Power BI  ·  ⛔ Phase 3 (designed, not yet built)

Executive analytics for HR leadership. Dashboards cover average onboarding duration, Day-1 Readiness rate, overdue task heatmaps, monthly volume trends, and probation outcome distribution. Embeds inside the model-driven app. Specified here; planned for Phase 3.

---

## Copilot Studio  ·  ⛔ Phase 3 (designed, not yet built)

Optional AI chatbot embedded in the new hire portal. Answers common questions ("When is my induction?", "What documents do I still need to upload?") grounded in the new hire's record data. Specified here; planned for Phase 3.

---

## Microsoft 365 Integrations

| Service | Use |
| --- | --- |
| Microsoft Teams | Task notifications and Adaptive Cards for in-Teams task completion |
| Outlook | Email notifications, reminders, escalations |
| SharePoint | Optional document storage at scale |

---

## Component-to-Requirement Mapping

| Component | Requirements Implemented |
| --- | --- |
| Dataverse | BR-02, BR-07, BR-12, NFR-01, NFR-04, NFR-06 |
| Model-Driven App | BR-01, BR-06, BR-10, NFR-02, NFR-05 |
| Power Pages | BR-14, NFR-01 (portal isolation) |
| Power Automate | BR-03, BR-04, BR-05, BR-08, BR-09, BR-11, BR-15, NFR-07 |
| Power BI | BR-13 |
| Copilot Studio | Optional enhancement to BR-14 |

---

➡️ Next: **[Section 9 — Process Flow Diagram](../09-Process-Flow-Diagram)**
