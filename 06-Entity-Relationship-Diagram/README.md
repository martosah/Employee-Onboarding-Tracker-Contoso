# 6. Entity Relationship Diagram

The data model consists of nine tables: four reference tables, four core transactional tables, and one supporting transactional table for documents.

![Employee Onboarding Tracker ERD](./erd_onboarding_tracker.png)

---

## Tables

### Department (reference)

| Column | Type | Notes |
| --- | --- | --- |
| Department Name | Single Line of Text | Primary Name |
| Department Code | Auto Number (DEP-001) | Unique |
| Department Head | Single Line of Text | |

### Job Role (reference)

| Column | Type | Notes |
| --- | --- | --- |
| Role Name | Single Line of Text | Primary Name |
| Role Code | Auto Number (JR-001) | Unique |
| Department | Lookup → Department | |
| Is Customer Facing | Yes/No | |

### Staff (reference)

| Column | Type | Notes |
| --- | --- | --- |
| Full Name | Single Line of Text | Primary Name |
| Staff Number | Auto Number (STF-001) | Unique (Alternate Key) |
| Staff Role | Choice | HRBP, Head of HR, IT Admin, Facilities Officer, Compliance Officer, Manager |
| Department | Lookup → Department | |
| Email | Email | |
| Backup Staff | Lookup → Staff | Self-reference |

### Task Template (reference)

| Column | Type | Notes |
| --- | --- | --- |
| Task Name | Single Line of Text | Primary Name |
| Template Code | Auto Number (TPL-001) | |
| Default Assignee Role | Choice | |
| Due Date Offset Days | Whole Number | Days relative to Start Date (negative = before) |
| Is Critical For Day 1 | Yes/No | |
| Applies To Job Role | Lookup → Job Role | Blank = all roles |
| Requires Equipment | Yes/No | |

### Employee (transactional)

| Column | Type | Notes |
| --- | --- | --- |
| Full Name | Single Line of Text | Primary Name |
| Employee Number | Auto Number (EMP-001) | Unique (Alternate Key) |
| Job Role | Lookup → Job Role | |
| Department | Lookup → Department | |
| Start Date | Date Only | |
| Reporting Manager | Lookup → Staff | |
| Employment Type | Choice | Full-time, Contract, Part-time |
| Work Location | Single Line of Text | |
| Personal Email | Email | |
| Work Email | Email | Populated by IT |

### onboarding record (transactional)

| Column | Type | Notes |
| --- | --- | --- |
| Case Number | Auto Number (CASE-001) | Primary Name |
| Employee | Lookup → Employee | Unique (1:1) |
| Case Status | Choice | Not Started, In Progress, At Risk, Completed |
| Case Opened Date | Date and Time | |
| Case Closed Date | Date and Time | |
| Assigned HRBP | Lookup → Staff | |
| Day-1 Readiness | Choice | Pending, Ready, At Risk |
| Probation Outcome | Choice | Passed, Extended, Failed |

### Onboarding Task (transactional)

| Column | Type | Notes |
| --- | --- | --- |
| Task Number | Auto Number (TSK-001) | Primary Name |
| Case | Lookup → onboarding record | Parental |
| Task Name | Single Line of Text | |
| Assigned To | Lookup → Staff | |
| Due Date | Date Only | |
| Status | Choice | Not Started, In Progress, Completed |
| Completed Date | Date and Time | |
| Completed By | Lookup → Staff | |
| Is Critical For Day 1 | Yes/No | |
| Notes | Multi-line Text | |
| Template | Lookup → Task Template | |

### Equipment (transactional)

| Column | Type | Notes |
| --- | --- | --- |
| Serial Number | Single Line of Text | Primary Name (Unique) |
| Equipment Type | Choice | Laptop, Monitor, Headset, Phone, Access Card, Other |
| Assigned To | Lookup → Employee | |
| Date Issued | Date Only | |
| Issued By | Lookup → Staff | |
| Related Task | Lookup → Onboarding Task | |

### New Hire Document (transactional)

| Column | Type | Notes |
| --- | --- | --- |
| Document Title | Calculated | Employee Full Name + " — " + Document Type |
| Case | Lookup → onboarding record | Parental |
| Document Type | Choice | BVN Slip, NIN Slip, Academic Certificate, Bank Details, Other |
| File Attachment | File | |
| Uploaded Date | Date and Time | |
| Status | Choice | Pending Review, Approved, Rejected |

---

## Relationships

| Parent | Child | Cardinality | Behaviour |
| --- | --- | --- | --- |
| Department | Job Role | 1:N | Referential, Restrict Delete |
| Department | Employee | 1:N | Referential, Restrict Delete |
| Department | Staff | 1:N | Referential, Restrict Delete |
| Job Role | Employee | 1:N | Referential, Restrict Delete |
| Job Role | Task Template | 1:N | Referential, Remove Link |
| Staff | Employee (Manager) | 1:N | Referential |
| Staff | onboarding record (HRBP) | 1:N | Referential |
| Staff | Onboarding Task (Assignee) | 1:N | Referential |
| Staff | Onboarding Task (Completed By) | 1:N | Referential |
| Staff | Equipment (Issued By) | 1:N | Referential |
| Staff | Staff (Backup) | 1:1 | Referential, self-reference |
| Employee | onboarding record | 1:1 | Referential, unique constraint |
| Employee | Equipment | 1:N | Referential, Restrict Delete |
| onboarding record | Onboarding Task | 1:N | Parental |
| onboarding record | New Hire Document | 1:N | Parental |
| Task Template | Onboarding Task | 1:N | Referential, Remove Link |
| Onboarding Task | Equipment | 1:0..1 | Referential |

---

```

---

➡️ Next: **[Section 7 — Business Logic](../07-Business-Logic)**
