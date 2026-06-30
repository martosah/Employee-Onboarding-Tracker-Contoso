# 16. Solution Hygiene and ALM

A build isn't finished when it works in the development environment — it's finished when it can be packaged and deployed cleanly somewhere else. This section documents the application lifecycle management (ALM) work: cleaning the solution of foreign components, resolving Solution Checker findings, exporting deployable artifacts, and the deployment path a production org would use.

This work surfaced a real, instructive problem, which is recorded honestly because it's one of the most common causes of failed Power Platform handoffs.

> **Two solutions, one publisher.** The build is packaged as two solutions under the Platform Explorers publisher: **Employee Onboarding Tracker** (the backend — tables, flows, app, BPF, security) and **Employee Onboarding Portal** (the Power Pages site). Part 1 of this section covers the Tracker's hygiene and its initial export; the final part covers the v1.1.0.0 bump and the portal export added in Phase 2 Part 2.

---

## The Preferred-Solution Trap

The Employee Onboarding Tracker solution had been set as the environment's **preferred solution**. The preferred solution is the environment's catch-all: any new or modified unmanaged component created *anywhere* in the environment is automatically added to it.

The same Developer environment also hosts other projects (security, compliance, asset management) with their own dataflows that run and refresh on a schedule. Because the onboarding solution was preferred, every one of those dataflow runs auto-attached the dataflow to it — along with foreign security roles, Dataverse Search index components, scheduled-refresh objects, and other items that had nothing to do with onboarding.

The telltale symptom: removing the foreign components didn't stick. They reappeared the next time the solution was opened, because the next dataflow run re-collected them. The lesson is that you can't win this by removing harder — you have to stop the collection at its source.

---

## The Fix

The resolution had two parts:

1. **Stop the collection.** The preferred solution was changed to the Common Data Services Default Solution, returning the catch-all behaviour to where it belongs. From that point, nothing new auto-attached to the onboarding solution, so removals finally held.
2. **Remove the foreign components once.** The orphaned dataflows, Dataverse Search index entries, and scheduled-refresh objects were removed from the solution — using *Remove from this solution*, never delete, so other projects' components were untouched.

One component that looked foreign was correctly **kept**: the MCP Server object carried the `pex_` prefix and belonged to this app, not another project. The cleanup brought the component count down from over a hundred to a clean set containing only the onboarding solution's own objects.

---

## Solution Checker — Part 1 History

Solution Checker was run iteratively, and the issue count was driven down deliberately rather than ignored:

| Run | Issues | What they were | Resolution |
| --- | :---: | --- | --- |
| First | 6 (Medium) | A BPF segmentation warning, plus five missing-dependency errors (a stray connection reference and four foreign security roles) | The foreign dependencies were carried by the foreign dataflows |
| After cleanup | 2 (Medium) | BPF segmentation warning + one stray connection reference | Removing the dataflows cleared the four foreign-role dependencies |
| Final (v1.0.0.0) | 1 (Medium) | BPF segmentation warning only | The stray reference was traced to one flow and repointed to the proper Dataverse connection reference |

The distinction that mattered throughout: *missing-dependency* errors genuinely fail an import and had to be fixed; a *segmentation* warning on a working component does not. The remaining single Medium was a non-blocking segmentation note on the business process flow entity — documented as a known low-severity item rather than chased.

---

## Export — Part 1 (Tracker v1.0.0.0)

The Tracker solution was first exported at version **1.0.0.0** in both forms:

- **Unmanaged** — the editable source and rollback copy. Re-importable into a development environment; this is what you'd branch or rebuild from.
- **Managed** — the deployable artifact. Components import locked, and the package supports clean upgrades and full uninstall. This is the file a target (test or production) environment would receive.

---

## Platform Dependencies

On export, Power Platform listed the **managed solutions this solution depends on** — Microsoft's own platform base solutions (Environment Variables, the Copilot infrastructure solutions, and Power Pages resources). This is not an error; it's a prerequisite manifest. A managed solution will only import into a target environment that already has these base solutions present at a compatible version.

In practice every standard Dataverse environment ships with them, so an import into a normal target succeeds. The failure case is a stripped-down or older target missing one of them. Recording this demonstrates an understanding of solution layering — the single most common reason a managed import fails in the real world.

A related note: this solution's flows read environment-variable **default values**, so the export did not prompt to strip current values (there were none to strip). In a multi-environment deployment, per-environment overrides would be set as current values in each target; the documented behaviour is that the flows read the default, so a target override requires updating the default value.

---

## Phase 2 Part 2 — Versioning and Packaging the Portal

When the Power Pages portal was built, the solution evolved — new flows, a new table, and the duplicate-prevention logic were added, and the flow inventory was standardised (number prefixes removed). That evolution was captured through **versioning rather than silent overwrite**, which is the discipline that distinguishes a maintained solution from a re-uploaded file.

**The Tracker was re-versioned to 1.1.0.0.** The v1.0.0.0 pair is retained in the repository as the part-one milestone (the build before duplicate prevention); v1.1.0.0 is the current build. A reader sees a genuine version progression — v1.0.0.0 → v1.1.0.0 — not two undated copies of the same number.

| Version | Adds |
| --- | --- |
| 1.0.0.0 | Initial build — data model, app, backend automation, BPF, security |
| 1.1.0.0 | Portal automation, the Rejected Document Archive table, duplicate prevention, and standardised flow naming |

**A fresh Solution Checker run validated the new version.** The earlier 6/15 result predated the portal-phase additions, so it could not certify what was being shipped. A re-run on v1.1.0.0 completed with **0 issues across all severities** (Critical 0, High 0, Medium 0, Low 0) — the earlier segmentation note did not recur. A zero-issue result dated to the same build it certifies is the artifact a reviewer can trust, so it is committed alongside the solution.

**Portal cleanup.** Before export, the portal site was cleaned of build scaffolding — orphaned "-deleted" web-page artifacts left over from page redesigns were removed so the exported site and the source-controlled files contain only live pages. Several apparent cleanup items proved to be non-issues on inspection (the live page set was already clean, the upload form's underlying Dataverse form was a working template rather than an orphan, and the test contact's data was already coherent), which is itself worth recording: *verify before deleting* is the safer ALM habit, because a delete doesn't roll back.

**The portal was exported as its own solution.** The Power Pages site is captured in the separate **Employee Onboarding Portal** solution — its web pages, web files, basic forms, lists, and table permissions. It was exported at **1.0.0.0** in both managed and unmanaged forms, and its own Solution Checker run also returned **0 issues**. Importing only the Tracker would deliver the backend with no portal; shipping both solutions makes the complete system reproducible in a target environment.

> **Source-control note (future enhancement).** For deeper version control of the portal, the Power Platform CLI (`pac powerpages download`) unpacks the site into editable YAML/HTML/JS that can be diffed and committed file-by-file. The solution export used here is the correct, sufficient deployment artifact; the CLI unpack is the documented next step for fine-grained source control.

---

## Deployment Path

There are two ways to move a solution between environments, and the choice reflects the maturity of the target organisation:

- **Manual export and import** *(used here)* — package to a `.zip`, carry it to the target, import by hand. Correct for a single-developer, single-environment portfolio build, and the universal artifact that can later be fed into any automated process. The Tracker is imported first (it carries the tables the portal binds to), then the Portal.
- **Power Platform Pipelines** *(the production pattern)* — an automated promotion from Dev → Test → Production with approvals, run history, and a consistent, auditable deployment. This is the enterprise standard, but it requires multiple environments to deploy *between*.

This project lives in a single Developer environment, so there is no target to pipeline into — manual export is the appropriate and correct mechanism. The documented production path is promotion via Pipelines (or Azure DevOps), with these managed solutions as the deployment units. Stating the correct enterprise pattern *and* why it wasn't instantiated here is more credible than implying a production pipeline ran from a lone development environment.

---

## Lessons

Three principles came out of this work and are worth stating plainly:

**Never leave a project solution set as the preferred solution.** The preferred solution is an environment-wide catch-all; a dedicated project solution set as preferred will silently accumulate every unmanaged change in the environment. Keep the default solution (or a disposable scratch solution) as preferred, and build *inside* the dedicated solution.

**A build is only as deployable as it is clean.** Foreign components and missing dependencies don't show up while you're working in the source environment — they surface as import failures in the target. Running Solution Checker and resolving real dependency errors before export is what separates a solution that demonstrates ALM from one that merely happens to run.

**Version, don't overwrite.** When a solution evolves, bump its version and keep the prior artifact. A version progression is auditable and tells a reviewer what changed and when; overwriting a same-named, same-versioned file erases that history.

---

## The Packaged Solutions

The exported solution files are in the [`/solution`](../solution) folder:

**Backend — Employee Onboarding Tracker**
- `EmployeeOnboardingTracker_1_1_0_0_managed.zip` — current deployable managed solution
- `EmployeeOnboardingTracker_1_1_0_0_unmanaged.zip` — current unmanaged source
- `EmployeeOnboardingTracker_1_0_0_0_*` — retained part-one milestone (v1.0.0.0)

**Portal — Employee Onboarding Portal**
- `EmployeeOnboardingPortal_1_0_0_0_managed.zip` — the deployable managed portal site
- `EmployeeOnboardingPortal_1_0_0_0_unmanaged.zip` — the unmanaged portal source

Import order in a fresh environment: **Tracker first, then Portal** (the portal binds to tables the Tracker provides). The portal then prompts for its connection references and environment-variable values, as documented in [`/solution`](../solution).

---

➡️ Next: **[Section 17 — Power Pages Portal: Architecture and Pages](../17-Power-Pages-Portal-Architecture-and-Pages)**
