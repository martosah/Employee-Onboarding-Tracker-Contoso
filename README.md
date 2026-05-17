# Employee Onboarding Tracker — Contoso

Phase 1 design and specification for an Employee Onboarding Tracker, produced as part of the **Platform Explorers Cohort 2 — Power Platform track (Week 4-6)**.

---

## The Problem

Contoso HR onboards 30+ new employees every month. IT setup, equipment issuance, and access provisioning are tracked in spreadsheets. Tasks fall through the cracks and new starters arrive without the laptops, accounts, and access they need to do their jobs.

The result is missed handoffs across HR, IT, Facilities, and Compliance, no clear ownership of tasks, no audit trail, and a poor first-day experience for every new hire.

---

## The Approach

I worked through the design in nine sections, each one feeding into the next:

| # | Section | Focus |
| --- | --- | --- |
| 1 | [Use Case and Domain](./01-Use-Case-and-Domain) | What the system is and what it covers |
| 2 | [Business Scenario](./02-Business-Scenario) | Contoso's pain points and the gaps behind them |
| 3 | [Roles in the Solution](./03-Roles-in-the-Solution) | The people who use the system |
| 4 | [Key Stakeholders](./04-Key-Stakeholders) | Internal and external stakeholders |
| 5 | [Business Requirements and Acceptance Criteria](./05-Business-Requirements-and-Acceptance-Criteria) | Functional and non-functional requirements with their definition of done |
| 6 | [Entity Relationship Diagram](./06-Entity-Relationship-Diagram) | The data model and relationships |
| 7 | [Business Logic](./07-Business-Logic) | The rules that govern system behaviour |
| 8 | [Power Platform Components](./08-Power-Platform-Components) | Components selected to deliver the solution |
| 9 | [Process Flow Diagram](./09-Process-Flow-Diagram) | End-to-end onboarding process by role |

---

## Key Design Choices

**One case per new hire.** Each new employee has exactly one Onboarding Case that parents all their tasks, equipment records, and uploaded documents.

**Tasks are generated from templates, not hard-coded.** A separate Task Template table holds the standard onboarding checklist. HR can change the checklist by editing template rows — no developer involvement.

**Role-tailored task generation.** Tasks tagged to a specific job role only generate for that role. Universal tasks generate for every new hire.

**Day-1 Readiness check.** Three days before the start date, the system flags any case where critical tasks are still incomplete and escalates to HR leadership.

**Self-service portal for new hires.** New hires upload personal documents (BVN, NIN, certificates) before Day 1 through an external-facing portal, removing manual chase-ups by HR.

**Probation milestones built in.** Onboarding doesn't close at Day 1. The case stays open through 30/60/90-day probation milestones owned by the reporting manager.

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
└── 09-Process-Flow-Diagram/
```

---

## Project Context

|  |  |
| --- | --- |
| **Programme** | Platform Explorers Cohort 2 |
| **Track** | Power Platform |
| **Use Case** | Employee Onboarding Tracker (Week 4-6) |
| **Author** | [Martins Osahon Osimen](https://github.com/martosah) |
| **Phase** | Phase 1 — Design and Specification |

---

## Acknowledgements

Thanks to the Platform Explorers programme and the cohort coaches — Juan Ojochemi Idowu, Mathew Ede, Rachel Irabor, Sarah Anueyiagu, Church Ephraim, Thomas Okuya, and Adewale Yusuf.

---

## License

MIT License — see [LICENSE](./LICENSE).
