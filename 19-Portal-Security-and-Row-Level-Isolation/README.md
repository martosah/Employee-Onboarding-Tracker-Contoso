# 19. Portal Security and Row-Level Isolation

The internal security model (Section 14) governs HR and operational staff through Dataverse security roles. New hires are governed by a completely separate mechanism: **Power Pages table permissions**, attached to **web roles**, scoped to the signed-in **contact**. This section documents how that model enforces the single most important portal guarantee — *a new hire sees and edits only their own onboarding record and documents, and never anyone else's*.

This is the portal's answer to acceptance criterion NFR-01.4 ("a new hire must only access their own record data") and BR-14.5 ("must NOT see internal staff tasks or other new hires' data").

---

## Two Separate Access Models

It is worth stating plainly why the portal can't simply reuse the Dataverse roles:

| | Internal staff | New hires |
| --- | --- | --- |
| Identity | Dataverse system user | Power Pages contact |
| Governed by | Security roles + Owner teams | Table permissions + web roles |
| Scope mechanism | Privilege depth (User / Org) | Contact / account relationship scope |
| Surface | Model-driven app | Power Pages portal |

The two never overlap. A new hire holds **no Dataverse security role at all**; their entire access surface is the portal, and everything they can see or do there is defined by table permissions. This separation is itself a security property — a portal user cannot reach the model-driven app or any data outside their table permissions, regardless of how the internal roles are configured.

---

## Contact-Scoped Table Permissions

The core control is a table permission with **Access type: Contact** on the New Hire Document table. "Contact" scope means a permission grants access only to rows related to the *signed-in contact* through a specified relationship — here, the **Uploaded By Contact** lookup (Section 11). So the permission reads, in effect: *"a hire may read and write New Hire Document rows where Uploaded By Contact is them."*

| Permission (parent) | Table | Access type | Relationship | Privileges |
| --- | --- | --- | --- | --- |
| New Hire – Own Documents | New Hire Document | **Contact** | Uploaded By Contact | Read, Write, Create, Append, Append To |
| New Hire – Own Onboarding | Onboarding Record | **Contact** | Portal Contact | Read |

Because access is scoped by the contact relationship rather than by ownership or a global role, two hires signed in at the same time each see a different, non-overlapping set of rows from the same table — the row-level isolation the requirement demands.

---

## Child-Permission Chains

A contact-scoped permission on a parent table doesn't automatically extend to *related* records — the document's file notes, or its rejected-document archive rows. Those need **child permissions** that inherit the parent's contact scope by walking a relationship.

This is where one Power Pages reality has to be stated firmly, because it is a common and costly misconception: **the Table Permissions UI has no "Access type: Parent" option.** You do not create a child permission by choosing "Parent" as an access type — that dropdown value does not exist. A child permission is created **from the parent permission's own "Child permissions" tab**, where you add a permission on the related table and choose the relationship that links it to the parent. The child then **inherits the parent's scope and web role automatically** — you do not (and cannot) set the role on the child directly.

The chains built under **New Hire – Own Documents**:

| Child permission | Table | Relationship to parent | Privileges |
| --- | --- | --- | --- |
| New Hire – Own Document Notes | Note (annotation) | Document → Notes | Read, Create, Append |
| New Hire – Read Own Rejected Archive | Rejected Document Archive | Document → Archive (`pex_originaldocument`) | Read |
| New Hire – Read Own Archive Notes | Note (annotation) | Archive → Notes | Read |

The result: a hire can read and attach files to their *own* documents, and can read the archived copy of their *own* rejected documents (which powers the "Your previous submission" card in Section 18) — all without ever being able to reach another hire's records, because every link in the chain inherits the same contact scope from the top.

---

## The Privilege Combinations That Make Attachments Work

Getting an *upload* to work through a portal edit form requires a specific, non-obvious set of privileges across the parent and the child note, and getting it wrong produces silent failures rather than clear errors:

- On the **parent** New Hire Document permission: **Write** *and* **Append To** — Append To is what allows a note to be attached *to* the document.
- On the **child** Note permission: **Create** *and* **Append** — Append is what allows the note to be attached, from the note's side.

Both halves are required. Missing Append To on the parent, or Append on the child, leaves the form able to display but unable to save an attachment. This pairing (Write + Append To on the parent, Create + Append on the child) is the working combination for edit-form attachments.

---

## Web Roles

Table permissions take effect only when attached to a **web role**, and a contact gets a web role by being a member of it. New hires are members of the **Authenticated Users** web role, which carries the table permissions above. The child permissions inherit this role from their parent — which is exactly why roles are never set on the children directly.

Because the site is **private** (Section 17), there is no Anonymous web role in play for onboarding data — every visitor is authenticated, and every authenticated visitor is scoped to their own rows by the contact-based permissions.

---

## What the Hire Cannot Reach

The isolation is defined as much by what's *absent* as by what's granted:

- **No access to Onboarding Task.** Internal staff tasks are never exposed to the portal — there is no table permission for them, so the "My Tasks" page was removed entirely (Section 17). A hire cannot see internal prep work.
- **No cross-hire visibility.** Every permission is contact-scoped, so a hire's queries return only their own rows. There is no permission that would return another hire's documents or record.
- **No write to status or review note.** A hire can upload and resubmit, but the approve/reject decision and the Review Note are set by HR in the model-driven app — the portal permissions grant the hire no path to change a document's review outcome.

---

## Verification

Isolation was confirmed by signing in as a real portal user (the test new hire, Bob Olise) and checking that:

- **My Documents showed only Bob's documents** — no rows belonging to any other hire, confirming the contact scope on New Hire Document.
- **The "previous submission" card resolved only Bob's archived files** — confirming the child-permission chain to the Rejected Document Archive inherited the same scope.
- **Uploads and resubmissions saved successfully** — confirming the Write + Append To / Create + Append combination was complete.
- **No internal task or other-hire data was reachable anywhere on the site** — confirming the absence of any broader permission.

The test matters because table-permission misconfigurations tend to fail *open* in subtle ways (a missing scope can expose more than intended) or *closed* (a missing privilege silently blocks a save). Verifying with a real signed-in contact, rather than assuming the configuration, is what confirms the boundary actually holds.

---

➡️ Next: **[Section 20 — Preventing Duplicate Documents](../20-Preventing-Duplicate-Documents)**
