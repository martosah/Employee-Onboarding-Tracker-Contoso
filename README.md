# Employee Onboarding Tracker — Contoso

An enterprise-grade onboarding solution built on the Microsoft Power Platform — taken from problem statement through design, build, and packaged deployment.

---

## The Problem

Contoso HR onboards 30+ new employees every month. IT setup, equipment issuance, and access provisioning are tracked in spreadsheets. Tasks fall through the cracks and new starters arrive without the laptops, accounts, and access they need to do their jobs.

The result is missed handoffs across HR, IT, Facilities, and Compliance, no clear ownership of tasks, no audit trail, and a poor first-day experience for every new hire.

---

## What This Project Delivers

A working Dataverse application that replaces the spreadsheets with a single source of truth: one onboarding record per new hire, tasks generated automatically from a configurable template library, role-based assignment and notification, scheduled readiness checks, and a guided process that runs from pre-boarding through the end of probation.

The project is documented in two phases:

- **Phase 1 — Design & Specification.** The problem, the people, the requirements and their acceptance criteria, the data model, the business rules, the component plan, and the end-to-end process flow. *(Sections 1–9.)*
- **Phase 2 — Building the Solution (Part 1).** The first components of the planned solution, built to solve the onboarding problem: the **Dataverse** data model, the **model-driven app** (the primary interface), and the **Power Automate** automation — plus the business process flow, security model, an operational metric (Preparation Lead Time), and the solution-hygiene and export work that makes it deployable. *(Sections 10–16.)* Remaining components, including the **Power Pages** self-service portal, follow in later phases. (see Build Status below).

---

## Build Status

Phase 1 designed the full target solution. Phase 2 built the data, app, and automation layers. The list below states exactly what is live versus what remains designed-but-not-yet-built, so the scope is unambiguous.

| Component | Designed (Phase 1) | Built (Phase 2) | Notes |
| --- | :---: | :---: | --- |
| Dataverse data model | ✅ | ✅ | 9 custom tables, relationships, keys, business rules |
| Model-Driven App | ✅ | ✅ | Forms, views, dashboards, site map |
| Power Automate | ✅ | ✅ | 17 cloud flows across generation, notification, scheduling, status, and BPF sync |
| Business Process Flow | ✅ | ✅ | Four-stage Onboarding Process |
| Security model | ✅ | ✅ | 4 security roles, depth-based access matrix |
| Managed/unmanaged solution export | — | ✅ | Packaged at v1.0.0.0 |
| **Power Pages portal** | ✅ | ⛔ **Not yet built** | New-hire self-service document upload — specified in Section 8, planned for a later phase |

---

## The Design (Phase 1)

Nine sections, each feeding into the next:

| # | Section | Focus |
| --- | --- | --- |
| 1 | [Use Case and Domain](01-Use-Case-and-Domain) | What the system is and what it covers |
| 2 | [Business Scenario](02-Business-Scenario) | Contoso's pain points and the gaps behind them |
| 3 | [Roles in the Solution](03-Roles-in-the-Solution) | The people who use the system |
| 4 | [Key Stakeholders](04-Key-Stakeholders) | Internal and external stakeholders |
| 5 | [Business Requirements and Acceptance Criteria](05-Business-Requirements-and-Acceptance-Criteria) | Functional and non-functional requirements with their definition of done |
| 6 | [Entity Relationship Diagram](06-Entity-Relationship-Diagram) | The data model and relationships |
| 7 | [Business Logic](07-Business-Logic) | The rules that govern system behaviour |
| 8 | [Power Platform Components](08-Power-Platform-Components) | Components selected to deliver the solution |
| 9 | [Process Flow Diagram](09-Process-Flow-Diagram) | End-to-end onboarding process by role |

## The Build (Phase 2)

Seven sections documenting the implemented solution:

| # | Section | Focus |
| --- | --- | --- |
| 10 | [Build Overview and Environment](10-Build-Overview-and-Environment) | Environment, publisher, and full component inventory |
| 11 | [Data Model as Built](11-Data-Model-as-Built) | How the ERD became real tables, relationships, and ownership rules |
| 12 | [Automation and Flows](12-Automation-and-Flows) | The 17 flows, grouped by purpose, with key engineering decisions |
| 13 | [Business Process Flow](13-Business-Process-Flow) | The four-stage Onboarding Process and its stage-gating logic |
| 14 | [Security Model](14-Security-Model) | The access matrix, privilege-depth reasoning, and design trade-offs |
| 15 | [Preparation Lead Time](15-Preparation-Lead-Time) | An operational metric, at-intake risk flagging, and how it is used |
| 16 | [Solution Hygiene and ALM](16-Solution-Hygiene-and-ALM) | Solution cleanup, Solution Checker results, export, and deployment path |

The packaged solution (managed and unmanaged, v1.0.0.0) is in [`/solution`](solution).

---

## Key Design Choices

**One Onboarding record per new hire.** Each new employee has exactly one Onboarding record that parents all their tasks, equipment records, and uploaded documents.

**Tasks are generated from templates, not hard-coded.** A separate Task Template table holds the standard onboarding checklist. HR can change the checklist by editing template rows — no developer involvement.

**Role-tailored task generation.** Tasks tagged to a specific job role only generate for that role. Universal tasks generate for every new hire.

**Day-1 Readiness check.** Three days before the start date, the system flags any onboarding record where critical tasks are still incomplete and escalates to HR leadership.

**Preparation Lead Time as an early-warning signal.** The system measures the days between a record being opened and the hire's start date, and flags compressed timelines the moment a record is created — not only as the start date approaches.

**Probation milestones built in.** Onboarding doesn't close at Day 1. The record stays open through 30/60/90-day probation milestones owned by the reporting manager.

**Self-service portal for new hires (designed, not yet built).** New hires will upload personal documents before Day 1 through an external-facing Power Pages portal. This is specified in Section 8 and planned for a later phase.

---

## Repository Structure

```
Employee-Onboarding-Tracker-Contoso/
│
├── README.md
├── LICENSE
│
├── 01-Use-Case-and-Domain/
├── 02-Business-Scenario/
├── 03-Roles-in-the-Solution/
├── 04-Key-Stakeholders/
├── 05-Business-Requirements-and-Acceptance-Criteria/
├── 06-Entity-Relationship-Diagram/
├── 07-Business-Logic/
├── 08-Power-Platform-Components/
├── 09-Process-Flow-Diagram/
│
├── 10-Build-Overview-and-Environment/
├── 11-Data-Model-as-Built/
├── 12-Automation-and-Flows/
├── 13-Business-Process-Flow/
├── 14-Security-Model/
├── 15-Preparation-Lead-Time/
├── 16-Solution-Hygiene-and-ALM/
│
└── solution/
```

---

## Project Context

|  |  |
| --- | --- |
| **Use Case** | Employee Onboarding Tracker |
| **Platform** | Microsoft Power Platform (Dataverse, Power Apps, Power Automate) |
| **Author** | [Martins Osahon Osimen](https://github.com/martosah) |
| **Phases** | Phase 1 — Design & Specification · Phase 2 — Build & Delivery |

---

## License

MIT License — see [LICENSE](LICENSE).
