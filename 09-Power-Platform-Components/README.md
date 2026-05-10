# 9. Power Platform Components

The solution uses six Power Platform components, supported by Microsoft 365 services for communication.

---

## Dataverse

The data foundation. Stores the nine tables, relationships, calculated columns, business rules, security roles, and audit configuration. Implements form-level rules and uniqueness constraints that do not require workflows.

---

## Power Apps — Model-Driven App

The primary interface for HR and operational staff. Hosts the New Hire Form, case detail forms, task forms, dashboards, and views. Used by HR Business Partners, Head of HR, IT Administrators, Facilities Officers, Compliance Officers, and Reporting Managers.

---

## Power Pages

The self-service portal for new hires. Hosts secure document upload, case status visibility, and Day-1 logistics. Used by new hires before they have corporate accounts.

---

## Power Automate

Implements the workflow automation. Three flow types are used:

- **Automated flows** triggered by Dataverse events: case creation, task generation, task notification, equipment validation, completion metadata, auto-closure, document handling.
- **Scheduled flows** running daily: pre-due reminders, day-of reminders, escalations, three-day and one-day readiness checks.
- **Manual flows** triggered from forms: case reassignment and ad-hoc readiness re-runs.

---

## Power BI

Executive analytics for HR leadership. Dashboards cover average onboarding duration, Day-1 Readiness rate, overdue task heatmaps, monthly volume trends, and probation outcome distribution. Embeds inside the model-driven app.

---

## Copilot Studio

Optional AI chatbot embedded in the new hire portal. Answers common questions ("When is my induction?", "What documents do I still need to upload?") grounded in the new hire's case data.

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
| Power Pages | BR-14 |
| Power Automate | BR-03, BR-04, BR-05, BR-08, BR-09, BR-11, BR-15, NFR-07 |
| Power BI | BR-13 |
| Copilot Studio | Optional enhancement to BR-14 |

---

⬅️ Back to: **[Main README](../README.md)**
