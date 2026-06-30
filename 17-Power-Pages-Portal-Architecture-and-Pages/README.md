# 17. Power Pages Portal: Architecture and Pages

Phase 2 Part 1 built the internal solution — the data, the model-driven app, and the automation that HR and operational staff use. Part 2 builds the part of the system that faces *outward*: a **Power Pages** self-service portal where a new hire, before they have any corporate account, can sign in, see how their onboarding is progressing, and upload the documents HR needs.

This section covers the portal's architecture — how a new hire becomes an authenticated user, the pages they see, how those pages show live data from Dataverse, and the branding that makes it feel like Contoso rather than a default template. The document workflow that runs *on* these pages, the security that isolates one hire's data from another's, and the duplicate-prevention logic are covered in Sections 18, 19, and 20 respectively.

---

## The Site

The portal is a Power Pages site — **Contoso New Hire Portal** — provisioned in the same Developer environment as the rest of the solution, so it reads and writes the same Dataverse tables directly with no integration layer in between. It is packaged as its own solution (**Employee Onboarding Portal**) for deployment, separate from the backend Tracker solution (see Section 16).

| Property | Value |
| --- | --- |
| Site name | Contoso New Hire Portal |
| Visibility | **Private** — only invited, authenticated contacts can view the site |
| Audience | New hires, before Day 1, before they have corporate credentials |
| Data source | The live Dataverse tables (New Hire Document, Onboarding Record, Rejected Document Archive) |
| Branding | Contoso — primary `#1857A8`, deep header `#0D3B66` |

Keeping the site **private** is a deliberate security posture, not a convenience. The portal exposes personal onboarding data, so anonymous access is never appropriate; every visitor must be a known contact who was explicitly invited.

---

## How a New Hire Becomes a User

A new hire is a person, not a system account — so the first problem the portal has to solve is identity. The solution links each new hire to a Dataverse **Contact** record, and that contact *is* their portal login.

The chain works like this:

1. **Provisioning.** The *Provision New Hire Portal Access* flow creates or links a Contact for the new hire and writes it to the **Portal Contact** lookup on their Onboarding Record. This binds the human to a portal identity.
2. **Invitation.** The new hire receives a registration email containing a deep link that lets them set a password and activate their account. The link is built with a `concat` expression that deliberately omits the `returnurl` parameter — an early version included it, and the ampersand encoding broke the invitation token. Removing it produced a clean, working activation link.
3. **Sign-in.** From then on the contact authenticates against the portal, and every page they load is scoped to *their* records through the Portal Contact relationship (the mechanism is detailed in Section 19).

The result is that "new hire Bob Olise" and "portal user Bob Olise" are the same identity, and the Onboarding Record knows which portal contact owns it.

---

## The Pages

The portal is intentionally small. A new hire has exactly three things to do — understand their status, see what to upload, and act — so the navigation carries three functional pages plus the standard system pages. Two further pages exist but are hidden from navigation because they are reached by clicking a document, not by browsing.

| Page | In nav? | Purpose |
| --- | :---: | --- |
| **Home** | ✅ | Personalised landing page — greeting, live progress, document checklist, and call-to-action buttons |
| **My Onboarding Status** | ✅ | Read-only view of how onboarding is progressing — the process-stage banner and document-approval progress |
| **My Documents** | ✅ | The new hire's action centre — the list of their documents with status, and the upload form |
| **Resubmit Document** | ⛔ hidden | Reached from a *rejected* document; lets the hire upload a corrected file against the same record (Section 18) |
| **Document Details** | ⛔ hidden | Reached by clicking a document row; shows the file and its review note read-only |
| Contact us / Access Denied / Page Not Found / Profile / Search | — | Standard Power Pages system pages |

A key design decision sits behind this shape. The internal solution has an **Onboarding Task** table, and an early portal draft exposed a "My Tasks" page. But every task in the system is *internal staff prep* (IT provisioning, facilities setup) — none is performed by the new hire. The new hire's only actions are the **documents they upload**. So "My Tasks" was removed from the portal entirely: surfacing internal tasks to a new hire would have been both confusing and a data leak. **My Documents is the new hire's to-do list**, and it is the only action surface they need.

---

## Showing Live Data

The pages are not static. Each one renders current data straight from Dataverse using **Liquid** templating with **FetchXML** queries, embedded in the page's `webpage.copy.html` and edited through the VS Code (vscode.dev) Power Pages workspace.

What each page renders live:

- **Home** — a personalised greeting using the signed-in user's name (`user.fullname`); a progress strip; and a "My Documents" panel that lists each required document with a tick when it is approved. This replaced an earlier static image, so the panel now reflects the hire's *actual* progress.
- **My Onboarding Status** — a four-stage **business-process-flow journey banner** (a pill bar showing Pre-Boarding → Day 1 Preparation → Onboarding In Progress → Completed, with the current stage highlighted), and a **document-approval progress card** that counts approved required documents against a denominator of five.
- **My Documents** — a live list of the hire's documents (title, type, status, uploaded date, review note) above the upload form.

**The required-document set** is fixed at five — Passport Photo, NIN Slip, BVN Slip, Academic Certificate, Bank Details — which is the denominator behind every "x of 5 approved" progress indicator. The **"Other"** document type is deliberately excluded from this count: it is a repeatable catch-all, not a required item, so it neither counts toward completion nor blocks it.

---

## Engineering Decisions

A few portal-specific platform behaviours shaped how these pages had to be built. They are recorded because, like the automation learnings in Section 12, they are not obvious until you hit them.

**Editing `webpage.copy.html` cleanly.** Partial edits to a page's HTML through the browser-based VS Code workspace were error-prone — a half-applied change could leave the markup broken. The reliable technique that emerged was to select all (`Ctrl+A`), delete, and paste the complete file, so every save is a known-good whole-file write rather than a risky splice.

**Read-only Choice fields render hidden option lists.** A read-only Choice column on a Power Pages form renders as a hidden `<select>` containing *all* of its option labels, not just the selected one. Any client-side script that reads the visible status therefore has to skip the hidden `<option>` elements, or it picks up every possible value at once. This mattered for the conditional logic on the Document Details and upload pages.

**Hero layout order, not reverse.** The Home and status pages use two-column "hero" panels (text beside a card). An initial implementation used `flex-direction: row-reverse` to put the card first visually while keeping the text first in the markup — but that caused a persistent column-stacking bug on mobile. The fix was to write the columns in **natural document order** (text first, card second) with plain `flex-direction: row`, which renders correctly on both desktop and mobile. The lesson: don't fight source order with `row-reverse` when responsive stacking is in play.

---

➡️ Next: **[Section 18 — Document Lifecycle and Portal Automation](../18-Document-Lifecycle-and-Portal-Automation)**
