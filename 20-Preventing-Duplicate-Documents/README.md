# 20. Preventing Duplicate Documents

The portal upload form lets a new hire pick a document type and submit a file. Left unguarded, nothing stops a hire submitting a second **NIN Slip** when one is already approved — spawning a duplicate row that HR has to review and reconcile. This section documents how that is prevented, using two complementary layers: a **client-side** guard that stops the duplicate being offered, and a **server-side** guard that catches it if the client is bypassed.

The two-layer approach is deliberate. The client layer gives a clean user experience (the hire never sees a type they shouldn't pick); the server layer gives genuine integrity (a determined user editing the page in dev tools still can't create a duplicate). Either alone is insufficient; together they are defense-in-depth.

---

## The Rule

The guard enforces one rule: **a new hire may submit each required document type at most once.**

The precise wording matters, because it determines what is blocked and what isn't:

- A document type that the hire has **already submitted in any status** — Approved, Pending Review, *or* Rejected — is **not offered again** on the upload form.
- A **Rejected** document is corrected through the **Resubmit page** (Section 18), which updates the *existing* row — it does **not** go back through a fresh upload. So a rejected type stays out of the upload dropdown by design; the correction path is the resubmit channel, not a new submission.
- **"Other"** is exempt. It is a repeatable catch-all (a hire may legitimately submit two different "Other" documents — a reference letter and a medical form, say), so it is never blocked.

This was a deliberate refinement during the build. An earlier framing blocked only *Approved* and *Pending* types and left *Rejected* available for fresh upload — but that would have produced two rows for a rejected type (the old rejected one plus a new upload), the exact duplicate being eliminated. Routing all corrections through resubmit, and blocking *every* already-submitted type, is the correct rule.

---

## Layer 1 — Client-Side Dropdown Guard

The upload form is a Power Pages basic form, so its Document Type dropdown is rendered at runtime — it isn't in the page HTML and can't be edited there. The guard is therefore **JavaScript**, placed in the My Documents page's `customjs.js` file.

It works by reading data already on the page rather than issuing new queries:

1. It reads the **My Documents list** already rendered above the form, collecting the Document Type of every row the hire has submitted.
2. It locates the upload form's Document Type `<select>`.
3. It **removes** every option whose label matches an already-submitted type — leaving only types the hire hasn't filed yet (plus the "Select" placeholder).
4. **"Other" is explicitly skipped**, so it remains selectable however many times it's been used.

Reading the on-page list (rather than a fresh FetchXML/Web API call) is what keeps this layer simple and dependency-free — no extra table permissions, no status-integer lookups, nothing to misconfigure. A short retry loop handles the list still loading when the script first runs.

The behaviour was confirmed on both desktop and mobile: for a hire who has submitted all five required types, the dropdown collapses to just the "Select" placeholder and "Other" — every required type correctly stripped, including the approved ones.

---

## Layer 2 — Server-Side Duplicate Catcher

The client guard prevents the duplicate being *offered*, but a determined user could re-add an option through browser dev tools and post anyway. The **Duplicate Document Catcher** flow (Section 18) closes that gap.

Its design is deliberately non-destructive, and reuses the lifecycle already built:

1. **Trigger** on New Hire Document **created**.
2. **Delay briefly** to let the portal's two-step attachment commit settle (Section 18) before evaluating — acting instantly would risk racing the file patch.
3. **Skip** the check for **"Other"** documents and for rows with no contact (which can't be checked).
4. **List** other documents of the *same type* for the *same contact*, excluding the new row itself — the filter built with `concat` for the dynamic contact and type values.
5. If at least one already exists, **update the new row to Rejected** with an explanatory Review Note ("Duplicate submission detected — you already have a [Type] document on file… use the Resubmit option rather than uploading a new one").

The decision to **reject rather than delete** is what makes this safe and clean. Rejecting reuses the existing flows: *Notify New Hire on Review Outcome* emails the hire the duplicate message, and *Archive Rejected Document & Free Slot* archives the file and frees the slot — so the duplicate is handled by machinery that already exists, with the Catcher only deciding *duplicate or not*. Deleting the row at creation time would have risked racing the file patch and would have bypassed the audit trail.

---

## Why Both Layers

Stated plainly, because the division of labour is the point:

| Layer | Prevents | Strength | Weakness |
| --- | --- | --- | --- |
| Client dropdown guard | The duplicate being *offered* | Clean UX — the hire never sees a disallowed type | Bypassable via dev tools |
| Server catcher flow | The duplicate being *persisted* | True integrity — runs regardless of the client | Acts after creation, not before |

The client layer is the everyday experience; the server layer is the guarantee. An enterprise control needs the guarantee, and a usable portal needs the experience — so the build ships both.

**One accepted residual**, documented rather than hidden: after a deliberate client bypass, the auto-rejected duplicate row is technically in a Rejected state, so it could in principle be resurrected through the Resubmit button. That only arises *after* someone has already tampered past the dropdown guard, and HR can reject it again manually — so it is a negligible edge case, recorded as a known limitation rather than engineered against.

---

## Phase 2 Part 2 — Complete

With duplicate prevention in place, the Power Pages portal is functionally complete: a new hire can sign in, see their progress, upload each required document once, receive a clear approve/reject outcome, correct a rejected document through resubmission, and never create a duplicate — all while seeing only their own data. Together with the internal solution from Part 1, the Employee Onboarding Tracker now spans the full Power Platform stack — **Dataverse, a model-driven app, Power Automate, and Power Pages** — built, secured, packaged, and deployable.

The two remaining specified components — **Power BI** dashboards and an optional **Copilot Studio** assistant (Section 8) — are reserved for **Phase 3**.

---

➡️ Return to the [project overview](../README.md).
