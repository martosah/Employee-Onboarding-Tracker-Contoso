# 16. Solution Hygiene and ALM

A build isn't finished when it works in the development environment — it's finished when it can be packaged and deployed cleanly somewhere else. This section documents the application lifecycle management (ALM) work: cleaning the solution of foreign components, resolving Solution Checker findings, exporting deployable artifacts, and the deployment path a production org would use.

This work surfaced a real, instructive problem, which is recorded honestly because it's one of the most common causes of failed Power Platform handoffs.

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

## Solution Checker

Solution Checker was run iteratively, and the issue count was driven down deliberately rather than ignored:

| Run | Issues | What they were | Resolution |
| --- | :---: | --- | --- |
| First | 6 (Medium) | A BPF segmentation warning, plus five missing-dependency errors (a stray connection reference and four foreign security roles) | The foreign dependencies were carried by the foreign dataflows |
| After cleanup | 2 (Medium) | BPF segmentation warning + one stray connection reference | Removing the dataflows cleared the four foreign-role dependencies |
| Final | 1 (Medium) | BPF segmentation warning only | The stray reference was traced to one flow and repointed to the proper Dataverse connection reference |

The remaining single Medium issue is a **segmentation note** on the business process flow entity — it warns that sub-components could be missed under certain managed-solution segmentation, but it does not block import and the BPF functions correctly. It is documented here as a known low-severity item rather than chased, which is the honest and proportionate call. The distinction that mattered throughout: *missing-dependency* errors genuinely fail an import and had to be fixed; a *segmentation* warning on a working component does not.

---

## Export — Managed and Unmanaged

The solution was exported at version **1.0.0.0** in both forms:

- **Unmanaged** — the editable source and rollback copy. Re-importable into a development environment; this is what you'd branch or rebuild from.
- **Managed** — the deployable artifact. Components import locked, and the package supports clean upgrades and full uninstall. This is the file a target (test or production) environment would receive.

Both are committed to the repository's [`/solution`](../solution) folder, so the project doesn't just *describe* the build — it *ships* it. A reviewer can download and import the managed solution.

---

## Platform Dependencies

On export, Power Platform listed the **managed solutions this solution depends on** — Microsoft's own platform base solutions (Environment Variables, the Copilot infrastructure solutions, and Power Pages resources). This is not an error; it's a prerequisite manifest. A managed solution will only import into a target environment that already has these base solutions present at a compatible version.

In practice every standard Dataverse environment ships with them, so an import into a normal target succeeds. The failure case is a stripped-down or older target missing one of them. Recording this demonstrates an understanding of solution layering — the single most common reason a managed import fails in the real world.

A related note: this solution's flows read environment-variable **default values**, so the export did not prompt to strip current values (there were none to strip). In a multi-environment deployment, per-environment overrides would be set as current values in each target; the documented behaviour is that the flows read the default, so a target override requires updating the default value.

---

## Deployment Path

There are two ways to move a solution between environments, and the choice reflects the maturity of the target organisation:

- **Manual export and import** *(used here)* — package to a `.zip`, carry it to the target, import by hand. Correct for a single-developer, single-environment portfolio build, and the universal artifact that can later be fed into any automated process.
- **Power Platform Pipelines** *(the production pattern)* — an automated promotion from Dev → Test → Production with approvals, run history, and a consistent, auditable deployment. This is the enterprise standard, but it requires multiple environments to deploy *between*.

This project lives in a single Developer environment, so there is no target to pipeline into — manual export is the appropriate and correct mechanism. The documented production path is promotion via Pipelines (or Azure DevOps), with this managed solution as the deployment unit. Stating the correct enterprise pattern *and* why it wasn't instantiated here is more credible than implying a production pipeline ran from a lone development environment.

---

## Lessons

Two principles came out of this work and are worth stating plainly:

**Never leave a project solution set as the preferred solution.** The preferred solution is an environment-wide catch-all; a dedicated project solution set as preferred will silently accumulate every unmanaged change in the environment. Keep the default solution (or a disposable scratch solution) as preferred, and build *inside* the dedicated solution.

**A build is only as deployable as it is clean.** Foreign components and missing dependencies don't show up while you're working in the source environment — they surface as import failures in the target. Running Solution Checker and resolving real dependency errors before export is what separates a solution that demonstrates ALM from one that merely happens to run.

---

## The Packaged Solution

The exported solution files are in the [`/solution`](../solution) folder:

- `EmployeeOnboardingTracker_1_0_0_0_managed.zip` — the deployable managed solution
- `EmployeeOnboardingTracker_1_0_0_0.zip` — the unmanaged source

---

➡️ Return to the [project overview](../README.md).
