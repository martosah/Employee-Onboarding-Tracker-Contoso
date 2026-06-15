# 6. Entity Relationship Diagram

The data model consists of nine tables: four reference tables, four core transactional tables, and one supporting transactional table for documents.

---

```mermaid
erDiagram
    DEPARTMENT ||--o{ JOB_ROLE : "groups"
    DEPARTMENT ||--o{ EMPLOYEE : "employs"
    DEPARTMENT ||--o{ STAFF : "employs"
    JOB_ROLE ||--o{ EMPLOYEE : "classifies"
    JOB_ROLE ||--o{ TASK_TEMPLATE : "scopes"
    STAFF ||--o{ STAFF : "backup"
    STAFF ||--o{ EMPLOYEE : "manages"
    STAFF ||--o{ ONBOARDING_RECORD : "owns as HRBP"
    STAFF ||--o{ ONBOARDING_TASK : "assigned / completed"
    STAFF ||--o{ EQUIPMENT : "issues"
    EMPLOYEE ||--o{ ONBOARDING_RECORD : "has (one per hire)"
    EMPLOYEE ||--o{ EQUIPMENT : "assigned"
    TASK_TEMPLATE ||--o{ ONBOARDING_TASK : "generates"
    ONBOARDING_RECORD ||--o{ ONBOARDING_TASK : "parents (cascade)"
    ONBOARDING_RECORD ||--o{ NEW_HIRE_DOCUMENT : "parents (cascade)"
    ONBOARDING_TASK ||--o{ EQUIPMENT : "relates"

    DEPARTMENT {
        guid DepartmentId PK
        string DepartmentName
        string DepartmentHead
    }
    JOB_ROLE {
        guid JobRoleId PK
        string RoleName
        lookup Department FK
    }
    STAFF {
        guid StaffId PK
        string FullName
        string WorkEmail
        lookup JobRole FK
        lookup Department FK
        lookup BackupStaff FK
        lookup AppUser FK
    }
    EMPLOYEE {
        guid EmployeeId PK
        string EmployeeNumber
        string FullName
        date StartDate
        lookup JobRole FK
        lookup Department FK
        lookup ReportingManager FK
        choice EmploymentType
        string WorkLocation
        string PersonalEmail
        string WorkEmail
    }
    TASK_TEMPLATE {
        guid TaskTemplateId PK
        string TaskName
        lookup AppliesToJobRole FK
        choice DefaultAssigneeRole
        int DueDateOffsetDays
        bool IsCriticalForDay1
        bool RequiresEquipment
    }
    ONBOARDING_RECORD {
        guid OnboardingRecordId PK
        string RecordNumber
        lookup Employee FK
        lookup AssignedHRBP FK
        choice RecordStatus
        choice Day1Readiness
        datetime RecordOpenedDate
        datetime RecordClosedDate
        choice ProbationOutcome
        int PreparationLeadTimeDays
        rollup TotalTasks
        rollup CompletedTasks
    }
    ONBOARDING_TASK {
        guid OnboardingTaskId PK
        string TaskNumber
        string TaskName
        lookup OnboardingRecord FK
        lookup TaskTemplate FK
        lookup AssignedTo FK
        lookup CompletedBy FK
        date DueDate
        choice Status
        bool IsCriticalForDay1
        date NewHireStartDate
        datetime CompletedDate
        string Notes
    }
    NEW_HIRE_DOCUMENT {
        guid NewHireDocumentId PK
        string DocumentTitle
        lookup OnboardingRecord FK
        choice DocumentType
        choice Status
        datetime UploadedDate
    }
    EQUIPMENT {
        guid EquipmentId PK
        string SerialNumber
        choice EquipmentType
        lookup AssignedTo FK
        lookup IssuedBy FK
        lookup RelatedTask FK
        date DateIssued
    }
```

> The Staff → Onboarding Task line covers two lookups (Assigned To and Completed By), shown as one relationship for readability; both appear as FKs on Onboarding Task. Parental relationships (Onboarding Record → Task and → Document) cascade on delete; all other lookups use Remove Link or Restrict per the as-built model.
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
| Record Number | Auto Number (REC-001) | Primary Name |
| Employee | Lookup → Employee | Unique (1:1) |
| Record Status | Choice | Not Started, In Progress, At Risk, Completed |
| Record Opened Date | Date and Time | |
| Record Closed Date | Date and Time | |
| Assigned HRBP | Lookup → Staff | |
| Day-1 Readiness | Choice | Pending, Ready, At Risk |
| Probation Outcome | Choice | Passed, Extended, Failed |

### Onboarding Task (transactional)

| Column | Type | Notes |
| --- | --- | --- |
| Task Number | Auto Number (TSK-001) | Primary Name |
| Onboarding Record | Lookup → onboarding record | Parental |
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
| Onboarding Record | Lookup → onboarding record | Parental |
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

➡️ Next: **[Section 7 — Business Logic](../07-Business-Logic)**
