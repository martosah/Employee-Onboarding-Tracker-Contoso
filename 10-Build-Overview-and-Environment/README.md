# 10. Build Overview and Environment

Phase 1 specified the solution. Phase 2 built it, in two parts: **Part 1** delivered the internal solution (Dataverse, the model-driven app, and the automation), and **Part 2** delivered the external-facing Power Pages portal. This section records *where* it was built, *under what publisher*, and *what the solution contains* — the foundation every later build section refers back to.

---

## Environment

The solution was built in a single Power Platform Developer environment. The discipline throughout was to build *as if* the solution would be promoted Dev → Test → UAT → Prod — using environment variables and connection references for every external dependency — even though only the development environment exists for this project.

| Property | Value |
| --- | --- |
| Environment name | Martins Projects |
| Type | Developer |
| Region | Europe |
| Dataverse | Provisioned |
| Auditing | Enabled (730-day retention) |

Auditing was switched on deliberately: one of the original pain points was the absence of an audit trail, so a tamper-evident change history is part of the delivered solution, not an afterthought.

---

## Publisher and Solutions

All components were created under a dedicated publisher so that every table, column, and choice carries a consistent prefix and the work is cleanly contained. The build is packaged as **two solutions** under that one publisher — the backend and the portal site — which are exported and deployed independently.

| Property | Value |
| --- | --- |
| Publisher | Platform Explorers |
| Customisation prefix | `pex` |
| Choice value prefix | `89107` |

| Solution | Contains | Version |
| --- | --- | --- |
| **Employee Onboarding Tracker** | The backend — tables, flows, model-driven app, business process flow, security roles, environment variables, connection references | **1.1.0.0** |
| **Employee Onboarding Portal** | The Power Pages site — web pages, web files, basic forms, lists, and table permissions (see Sections 17–20) | **1.0.0.0** |

The `pex` prefix appears on every custom object (for example `pex_onboardingrecord`, `pex_rejecteddocumentarchive`), which makes it immediately clear which components belong to this solution versus the platform's own. Why the portal is a separate solution — and how both are exported — is covered in [Section 16](../16-Solution-Hygiene-and-ALM).

---

## Component Inventory

The solution contains the following components. The data, app, automation, business process flow, and security model were built in **Phase 2 Part 1**; the Power Pages site, the Rejected Document Archive table, and the five portal flows were added in **Phase 2 Part 2**.

| Component type | Count | Detail |
| --- | --- | --- |
| Custom tables | 10 | Department, Job Role, Staff, Task Template, Employee, Onboarding Record, Onboarding Task, New Hire Document, Equipment, and Rejected Document Archive (Part 2) |
| Cloud flows | 22 | Task generation, notifications, scheduled checks, status automation, process-flow sync, and the portal document lifecycle (see Sections 12 and 18) |
| Power Pages site | 1 | Contoso New Hire Portal — the new-hire self-service portal (see Sections 17–20) |
| Business Process Flow | 1 | Onboarding Process — four stages (see Section 13) |
| Model-driven app | 1 | Employee Onboarding Tracker, with its site map |
| Security roles | 4 | HR Business Partner, Head of HR, Operations Staff, Reporting Manager (see Section 14), plus portal table permissions and web roles (see Section 19) |
| Environment variables | 6 | Externalised configuration (see below) |
| Connection references | 4 | Externalised connections (see below) |

---

## The Model-Driven App — Primary Interface

The model-driven app **Employee Onboarding Tracker** is the primary interface for HR and operational staff. It hosts the New Hire Form, record detail forms, task forms, dashboards, and views, organised into a site map with three working areas (Daily Work, People & Assets, Reference Data) plus a Home area for dashboards.

It serves every role in the process — HR Business Partners, Head of HR, IT Administrators, Facilities Officers, Compliance Officers, and Reporting Managers — with each role seeing a tailored view of the data. In the build, those business roles are implemented as four Dataverse security roles (HR Business Partner, Head of HR, Operations Staff, Reporting Manager); the operational roles (IT, Facilities, Compliance) are served through the shared Operations Staff role. The full access model is covered in [Section 14](../14-Security-Model).

New hires do not use the model-driven app — they interact with the solution through the **Power Pages portal** (Sections 17–20), which reads and writes the same Dataverse tables.

---

## Environment Variables

Environment variables hold configuration that would change between environments, so the same solution can be deployed elsewhere by editing values rather than editing flows.

| Variable | Purpose |
| --- | --- |
| Head of HR Email | Recipient for leadership escalations |
| HR Notifications Inbox | Shared mailbox used as the sender/recipient for system notifications |
| New Hire Portal URL | Link included in the welcome email to new hires |
| Day 1 Readiness Lead Days | Threshold (in days) used by the readiness check and at-intake risk flag |
| Enable Email Notifications | Master on/off switch for outbound email |
| All Contoso HR Members | HR distribution reference |

---

## Connection References

Connection references decouple flows from the specific connections they use, so promotion to another environment rebinds the connections without touching flow logic. All four are bound and used consistently across every flow, including the portal flows.

| Connection reference | Connector |
| --- | --- |
| Onboarding – Dataverse | Microsoft Dataverse |
| Onboarding – Outlook | Office 365 Outlook |
| Onboarding – Office 365 Users | Office 365 Users |
| Onboarding – Teams | Microsoft Teams |

---

## Identity and Team Model

User access is granted through **Dataverse Owner teams**, not direct per-user role assignments. Each team carries one security role, and members inherit it. After team inheritance was verified, the original per-user role assignments were removed so the team is the single source of access — the detail that makes the model genuinely team-based rather than redundant.

| Owner team | Security role | Members |
| --- | --- | --- |
| HR Business Partners | HR Business Partner | Megan Bowen, Grady Archie |
| HR Leadership | Head of HR | Nestor Wilke |
| Operations & IT | Operations Staff | Miriam Graham, Diego Siciliani, and operations members |
| Reporting Managers | Reporting Manager | Sandra Osimen |

In a full production setting this would be taken one step further — the Dataverse teams linked to Microsoft Entra security groups so that IT's existing group management governs membership. That is documented as the production pattern; the Owner-team model here demonstrates the same access outcome without the external dependency.

> **Portal identity is separate.** Internal staff authenticate as Dataverse users via the teams above. New hires authenticate as **portal contacts** through Power Pages, governed by table permissions and web roles rather than Dataverse security roles — a distinct access model covered in [Section 19](../19-Portal-Security-and-Row-Level-Isolation).

> **Note on user identity in automation.** Task ownership is set by reading each staff member's system-user identity from an **App User** lookup column on the Staff table, rather than calling the system-user metadata endpoint at run time. This design decision — and why it was made — is covered in Section 12.

---

## Project Status — A Continuing Build

This solution is a living project, not a finished product. **Phase 2 Part 1** delivered the data model, the model-driven app, the automation, the business process flow, and the security model. **Phase 2 Part 2** delivered the Power Pages self-service portal — document upload, HR review, resubmission, notifications, row-level isolation, and duplicate prevention. Both parts are built, packaged, and exported.

Two specified components remain for a future **Phase 3**: the **Power BI** executive dashboards and an optional **Copilot Studio** assistant (both specified in [Section 8](../08-Power-Platform-Components)). The repository will continue to grow as those components are built.

---

➡️ Next: **[Section 11 — Data Model as Built](../11-Data-Model-as-Built)**
