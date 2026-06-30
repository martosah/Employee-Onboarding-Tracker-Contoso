# 14. Security Model

Security in this solution is built on a single principle: **need-to-know, by role**. HR owns the onboarding process, Operations and IT execute the tasks, Reporting Managers supervise their own hires, and HR Leadership oversees everything. Each role sees and edits exactly what its job requires — no more.

This section covers the **internal** security model — the access enjoyed by HR and operational staff in the model-driven app, implemented with four Dataverse security roles, delivered through Owner teams, and tuned using privilege depths rather than custom code. The **portal** has a separate access model (table permissions and web roles for new hires), covered in [Section 19](../19-Portal-Security-and-Row-Level-Isolation).

---

## The Four Roles and Their Teams

Access is granted by **Owner team**, not by assigning roles to individuals. Each team carries one security role; members inherit it. After inheritance was verified, the original per-user role assignments were removed so the team is the single source of access.

| Owner team | Security role | Represents |
| --- | --- | --- |
| HR Business Partners | HR Business Partner | The HRBPs who own onboarding records |
| HR Leadership | Head of HR | Senior HR oversight |
| Operations & IT | Operations Staff | The people who perform IT, facilities, and equipment tasks |
| Reporting Managers | Reporting Manager | Line managers supervising their own new hires |

Every role also carries Note (Annotation) permissions at minimum, so attachments and document notes function for all users.

---

## Access Matrix

Depths are shown as **User** (own records only), **Org** (all records), or **None**.

| Table | HR Business Partner | Head of HR | Operations Staff | Reporting Manager |
| --- | --- | --- | --- | --- |
| Onboarding Record | Create/Read/Write — **User** | Read/Write — Org | Read — **Org** | Read — **Org** |
| Onboarding Task | Read/Write — **User** | Read/Write — Org | Read — Org, Write — **User** | Read — Org, Write — **User** |
| New Hire Document | Create/Read/Write — **User** | Read — Org | **None** | **None** |
| Employee | Read — Org | Read/Write — Org | Read — **Org** | Read — **Org** |
| Equipment | Read | Read | Create/Read/Write — Org | Read |
| Staff / Department / Job Role | Read | Read/Write | Read | Read |
| Task Template | Read | Read/Write | Read | Read |

---

## Understanding Privilege Depth

Every privilege is set at a **depth** that answers one question: *whose records does this apply to?* The ladder runs:

**None → User → Business Unit → Parent: Child BU → Organization**

Two contrasting examples from the build make this concrete:

- **HRBP, Read = User on Onboarding Record.** Megan Bowen sees only the records she owns. She cannot see Grady Archie's records. Same privilege, same table — but User depth scopes it to ownership.
- **Operations Staff, Read = Organization on Onboarding Record.** Miriam Graham sees every record in the environment, regardless of who owns it, because operations work spans all hires.

Read controls visibility; Write controls the ability to save changes — they are independent dials. Operations Staff have Read-Org but Write-None on Employee: they can see a hire's start date and work location (context for delivering a laptop) but cannot change a single field.

This environment uses a single root business unit, so Business-Unit depth would currently behave like Organization. Access differentiation is therefore achieved through privilege depth and Owner teams rather than a business-unit hierarchy — a deliberate, documented choice.

---

## Record and Task Isolation

**HRBP record isolation works through ownership plus parental cascade.** An HRBP has User-depth on Onboarding Record, Task, and Document, so they see only the records they own. Because Onboarding Record → Onboarding Task and → New Hire Document are **parental** relationships, access to a record cascades to its child tasks and documents. The net effect: an HRBP sees their own records *and everything beneath them*, but nothing belonging to another HRBP. (Inherited access materialises for records created after the relationship was set to parental — a known cascade behaviour.)

**Task isolation works through Write depth.** Operations Staff and Reporting Managers have Read-Org on tasks (they can see the whole queue) but Write-**User** (they can only save changes to tasks they own). Task ownership is set by the generation flow to the assignee (via the App User identity — Section 12), so Diego Siciliani can edit his own tasks but not Miriam's, even though he can see both.

---

## The Read-Organization Trade-Off

The most scrutinised decision in the model is giving Operations and Reporting Managers **Read-Org on Onboarding Record and Employee** rather than something narrower. This is a deliberate trade-off, documented rather than hidden:

**Why it's necessary.** Task views and the task form's Quick View join columns from the parent Onboarding Record and Employee (start date, lead time, readiness). A task assignee owns no parent records, so User depth returns nothing and the views error out. Dataverse role depths cannot express "the parent records of the tasks I own," so the workable depth is Organization.

**Why it's acceptable.** What Read-Org on these tables exposes is low-sensitivity *process context* — record number, status, dates, readiness. The genuinely confidential content is handled separately (below). Operations also benefits operationally from seeing the whole queue: a **shared-queue model** supports backup coverage (a colleague can cover an absent staff member's tasks) and task sequencing (an account can't be configured before it's created). Visibility is broad by design; control stays narrow through Write-User.

**The stricter alternative, noted for completeness.** The data could be denormalised onto the task so Operations needs no parent access at all. This was rejected as more engineering for marginal gain, given the Quick-View approach already surfaces context without duplicating data. It's recorded as a possible future enhancement.

---

## Confidential Data Handling

The model isolates sensitive content rather than relying on broad access being harmless:

- **New Hire Documents** (signed contracts, ID documents) are completely inaccessible to Operations and Reporting Managers — **None** on that table. This is where genuinely confidential material lives.
- **Probation Outcome** (on Onboarding Record) and **Personal Email** (on Employee) are flagged as **column-security candidates** — fields that should be readable only by HR even where the table itself is readable. Column security profiles are the documented tool for this; noting it demonstrates the field-level control exists even though it isn't enforced in this build.

---

## Dashboard Scoping

The HR Operations Dashboard surfaces process-health information (average lead time per HRBP, at-risk counts) that is management information, not operational data. It is scoped with security roles to **HR Business Partner, Head of HR, and System Administrator** only, so it doesn't appear at all for Operations or Reporting Manager users — cleaner than showing a dashboard that would otherwise throw a "no read privilege" error.

---

## Verification

The model was tested with real users, not just assumed:

- Megan Bowen saw only her owned records, not another HRBP's — confirming User-depth isolation.
- A task assignee saw only the tasks assigned to them for editing — confirming Write-User isolation.
- After granting Read-Org, Operations users' task views and Quick View rendered correctly, while New Hire Documents stayed hidden — confirming the trade-off works as designed.

---

## Production Hardening (Documented, Not Built)

In a full production tenant this model would be taken one step further: the Dataverse Owner teams would be linked to **Microsoft Entra security groups**, so that IT's existing group management governs team membership automatically. The Owner-team model used here demonstrates the same access outcomes without that external dependency, and the Entra-group linkage is the documented production pattern.

---

## Portal Security — A Separate Model

Everything above governs **internal** users — HR and operational staff — through Dataverse security roles. **New hires never receive a Dataverse security role.** They access the system only through the Power Pages portal, where access is governed by a different mechanism entirely: **table permissions** scoped to the signed-in contact, attached to **web roles**, enforcing that each hire sees and edits only their own onboarding record and documents.

This portal access model — the contact-scoped permissions, the child-permission chains for related notes and the rejected-document archive, and how row-level isolation is verified — is documented in [Section 19](../19-Portal-Security-and-Row-Level-Isolation). The two models are complementary: Dataverse roles secure the internal app, table permissions secure the portal, and the two never overlap because internal staff and new hires authenticate through entirely separate identity surfaces.

---

➡️ Next: **[Section 15 — Preparation Lead Time](../15-Preparation-Lead-Time)**
