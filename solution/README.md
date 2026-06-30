# Solution Packages

The exported Power Platform solutions for the Employee Onboarding Tracker. The build ships as **two solutions** under the Platform Explorers publisher: the **backend** (data, app, automation, BPF, security) and the **portal site** (Power Pages).

---

## Backend — Employee Onboarding Tracker

The Dataverse tables, the 22 cloud flows, the model-driven app, the business process flow, the security roles, the environment variables, and the connection references. Current version **1.1.0.0**; the v1.0.0.0 pair is retained as the part-one milestone (the build before the Power Pages portal).

| File | Type | Version | Use |
| --- | --- | --- | --- |
| `EmployeeOnboardingTracker_1_1_0_0_managed.zip` | Managed | 1.1.0.0 | Deploy into a test or production environment. Components import locked; supports clean upgrade and uninstall. |
| `EmployeeOnboardingTracker_1_1_0_0_unmanaged.zip` | Unmanaged | 1.1.0.0 | Editable source / rollback copy. Import into a development environment to modify or extend. |
| `EmployeeOnboardingTracker_1_0_0_0_managed.zip` | Managed | 1.0.0.0 | Retained part-one milestone (before the portal). |
| `EmployeeOnboardingTracker_1_0_0_0_unmanaged.zip` | Unmanaged | 1.0.0.0 | Retained part-one milestone (before the portal). |

| Version | Adds |
| --- | --- |
| 1.0.0.0 | Initial build — data model, model-driven app, backend automation, BPF, security |
| 1.1.0.0 | Portal automation, the Rejected Document Archive table, duplicate prevention, and standardised flow naming |

---

## Portal — Employee Onboarding Portal

The Power Pages site: web pages, web files, basic forms, lists, and table permissions for the new-hire self-service portal (see [Section 17](../17-Power-Pages-Portal-Architecture-and-Pages)). Version **1.0.0.0**.

| File | Type | Version | Use |
| --- | --- | --- | --- |
| `EmployeeOnboardingPortal_1_0_0_0_managed.zip` | Managed | 1.0.0.0 | Deploy the portal site into a test or production environment. |
| `EmployeeOnboardingPortal_1_0_0_0_unmanaged.zip` | Unmanaged | 1.0.0.0 | Editable portal source. Import into a development environment to modify or extend. |

---

## Importing

**Import order matters: deploy the backend first, then the portal.** The portal binds to Dataverse tables (New Hire Document, Onboarding Record, Rejected Document Archive) that the Tracker solution provides — importing the portal into an environment without them will fail.

1. In the target environment, go to **Solutions → Import solution**.
2. Import **Employee Onboarding Tracker** first (managed for test/production, unmanaged for development).
   - When prompted, bind the four connection references to connections in the target environment, and set environment-variable values appropriate to that environment.
3. Import **Employee Onboarding Portal** second.
   - Bind any connection references it prompts for and set its environment-variable values.
4. After import, confirm the Power Pages site provisions and the table permissions are active.

**Prerequisite:** the target environment must already have the standard Dataverse platform base solutions present (Environment Variables, Copilot infrastructure, Power Pages resources) at compatible versions — standard environments satisfy this by default. See [Section 16](../16-Solution-Hygiene-and-ALM) for the full dependency and deployment notes.

---

## Validation

Both solutions passed Solution Checker with **0 issues** (Critical / High / Medium / Low all zero) at the versions above. The checker result screenshots are recorded alongside the build documentation in [Section 16](../16-Solution-Hygiene-and-ALM).

> **Source-control note.** These `.zip` files are the deployable solution artifacts. For fine-grained version control of the portal specifically, the Power Platform CLI (`pac powerpages download`) can unpack the site into editable YAML/HTML/JS for file-by-file diffing — a documented future enhancement, not required for deployment.
