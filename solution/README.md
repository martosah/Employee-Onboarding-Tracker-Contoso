# Solution Package

The exported Power Platform solution for the Employee Onboarding Tracker, version **1.0.0.0**.

| File | Type | Use |
| --- | --- | --- |
| `EmployeeOnboardingTracker_1_0_0_0_managed.zip` | Managed | Deploy into a test or production environment. Components import locked; supports clean upgrade and uninstall. |
| `EmployeeOnboardingTracker_1_0_0_0.zip` | Unmanaged | Editable source / rollback copy. Import into a development environment to modify or extend. |

## Importing

1. In the target environment, go to **Solutions → Import solution**.
2. Upload the relevant `.zip` (managed for test/production, unmanaged for development).
3. When prompted, bind the four connection references to connections in the target environment, and set environment-variable values appropriate to that environment.

**Prerequisite:** the target environment must already have the standard Dataverse platform base solutions present (Environment Variables, Copilot infrastructure, Power Pages resources) at compatible versions — standard environments satisfy this by default. See [Section 16](../16-Solution-Hygiene-and-ALM) for the full dependency and deployment notes.
