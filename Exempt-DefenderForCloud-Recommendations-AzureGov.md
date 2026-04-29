# Exempt Microsoft Defender for Cloud recommendations in Azure Government using Azure Policy

In Azure Government, the **Exempt** action on a Microsoft Defender for Cloud (MDC) recommendation is **not yet available** in the portal (the feature exists in Azure commercial but has not been released to Azure Government). Until it is, the same outcome can be achieved by creating an **Azure Policy exemption** against the underlying policy assignment (the Microsoft Cloud Security Benchmark, or MCSB, initiative). Defender for Cloud honors those exemptions and shows the affected resources as **Not applicable** with the reason **Exempt** once compliance re-evaluates.

This article uses the recommendation **"Azure Backup should be enabled for virtual machines"** (built-in policy definition `013e242c-8828-4970-87b3-ab247555486d`, member of MCSB) and the scenario *"another (third-party) backup solution already protects these virtual machines"* to demonstrate creating exemptions at the **subscription**, **resource group**, and **individual resource** scopes.

## Cited resources

- [Azure Policy exemption structure](https://learn.microsoft.com/azure/governance/policy/concepts/exemption-structure)
- [Exempt resources from Defender for Cloud recommendations](https://learn.microsoft.com/azure/defender-for-cloud/exempt-resource)
- [Microsoft Cloud Security Benchmark (MCSB) initiative](https://learn.microsoft.com/security/benchmark/azure/introduction)
- [Built-in policy: Azure Backup should be enabled for virtual machines](https://learn.microsoft.com/azure/governance/policy/samples/built-in-policies#backup)
- [Resource Policy Contributor built-in role](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#resource-policy-contributor)
- [Security Admin built-in role](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#security-admin)

## Defender for Cloud "Exempt" capability — what we are reproducing

| Defender for Cloud (commercial) field | Description |
|---|---|
| Scope | Management group, subscription, or selected resources |
| Name | Friendly name for the exemption |
| Expiration date | Optional |
| Category | **Resolved through 3rd party (mitigated)** or **Risk accepted (waiver)** |
| Description / justification | Free text |

Azure Policy exposes the same fields, with two named categories: `Mitigated` and `Waiver`. See [Mitigated vs Waiver — choosing a category](#mitigated-vs-waiver--choosing-a-category) below for guidance.

## Field-to-property mapping

The portal fields above map to the underlying Azure Policy exemption properties (`name`, `displayName`, `expiresOn`, `exemptionCategory`, `policyDefinitionReferenceId`, `description`, `metadata`, `resourceSelectors`) documented in [Azure Policy exemption structure](https://learn.microsoft.com/azure/governance/policy/concepts/exemption-structure). No JSON authoring is required for this walkthrough.

## Prerequisites

### Confirm where the MCSB initiative is assigned

When Microsoft Defender for Cloud is enabled on a subscription, it automatically creates a policy assignment with the definition name **`SecurityCenterBuiltIn`** **at the subscription level**. The MCSB initiative is **not** auto-assigned at a management group, resource group, or individual resource.

> [!NOTE]
> The display name shown in the portal for this assignment has changed over time. Depending on tenant, region, and when the assignment was created, you may see any of the following — they all refer to the same underlying assignment (`SecurityCenterBuiltIn`):
>
> - **ASC Default (subscription: \<id\>)** — original display name.
> - **Defender Default** — newer display name (commonly seen in commercial).
> - **Microsoft cloud security benchmark** — current display name in many tenants.
>
> When following the steps below, match on the definition name `SecurityCenterBuiltIn` rather than the display name.

> [!IMPORTANT]
> An Azure Policy exemption must target an **existing assignment**. Before creating an exemption at a management group, resource group, or single-resource scope, confirm where the MCSB initiative is actually assigned and create the exemption at (or under) that scope. The default location is the **subscription**.
>
> - If MCSB is assigned only at the subscription (the default), create the exemption at the **subscription**, **resource group**, or **individual resource** scope — all three target the subscription-level `SecurityCenterBuiltIn` assignment.
> - If you (or your organization) have additionally assigned the MCSB initiative at a **management group**, you can create an exemption at that management group scope targeting the MG-level assignment.

**Quick check in Defender for Cloud (recommended):**

1. In the Azure Government portal, open **Microsoft Defender for Cloud** > **Environment settings** and select the subscription (or management group).
2. Select **Security policies**.
3. On the **Standards** tab, find the **Microsoft cloud security benchmark** row.
4. Read the **Assigned on** column — for example, **Subscription** (the default) or a management group name. That value is the scope where the MCSB assignment lives, and the scope at (or under) which you should create the exemption.

**Or via Azure Policy:** open **Policy** > **Assignments**, set the scope selector to the level you intend to use, and confirm the MCSB / `SecurityCenterBuiltIn` assignment is listed there (not just inherited from a higher scope).

### RBAC required to create an exemption

To create an Azure Policy exemption you need both an Azure RBAC role at the target scope **and** the `exempt/action` permission on the policy assignment. The built-in roles below grant both.

**Required Azure RBAC role at the exemption scope** (any one of):

- **Owner**
- **Resource Policy Contributor**
- **Security Admin**

The role must be assigned **at the same scope as the exemption (or higher)**:

- **Management group exemption** → role on the management group (or tenant root).
- **Subscription exemption** → role on the subscription (or parent MG).
- **Resource group exemption** → role on the resource group (or parent subscription / MG).
- **Single-resource exemption** → role on that resource (or any parent).

**Additional permission required for management group scope:**

- Assign the **Reader** role on the management group to the **Windows Azure Security Resource Provider** service principal. Without this, Defender for Cloud cannot read the exemption and the resource will not appear as **Not applicable / Exempt** in the MDC portal. (See [Exempt resources from Defender for Cloud recommendations — Before you start](https://learn.microsoft.com/azure/defender-for-cloud/exempt-resource#before-you-start).)

**Minimum granular permissions** (use these if you build a custom role instead of using a built-in role):

- `Microsoft.Authorization/policyExemptions/read`
- `Microsoft.Authorization/policyExemptions/write`
- `Microsoft.Authorization/policyExemptions/delete`
- `Microsoft.Authorization/policyAssignments/exempt/action`

> [!NOTE]
> Microsoft Entra directory roles (for example, Global Administrator or the Entra "Security Administrator" role) are **not** sufficient on their own — Azure RBAC at the resource scope is what is evaluated when creating an exemption.

## Scenario data used in the examples

| Field | Value |
|---|---|
| Recommendation | Azure Backup should be enabled for virtual machines |
| Built-in policy definition ID | `013e242c-8828-4970-87b3-ab247555486d` |
| Initiative | Microsoft Cloud Security Benchmark (`1f3afdf9-d0c9-4c3d-847f-89da613e70a8`) |
| Category | `Mitigated` (third-party backup product is in use) |
| Expiration | `2027-04-29T00:00:00Z` (1 year) |
| Justification | "Workloads protected by approved third-party backup solution. See change ticket CHG0012345." |

## Create the exemption (portal)

The same flow works for subscription, resource group, and single-resource scopes — only the **Exemption scope** value on the **Basics** tab differs.

### Permissions

- Owner / Resource Policy Contributor / Security Admin **at the scope where the exemption will live** (subscription, resource group, or individual resource).

### Steps

1. In the Azure Government portal, open the MCSB / `SecurityCenterBuiltIn` assignment:
   - **Subscription scope:** **Policy** > **Assignments** at the subscription, then open the assignment.
   - **Resource group or single-resource scope:** browse to the resource group (or resource) > **Policies** > **Assignments**, then open the inherited MCSB assignment.
2. Select **Create exemption** from the command bar.
3. On the **Basics** tab:
   - **Exemption scope:** select the **...** picker and choose the subscription, resource group, or specific resource you want exempted. The **Assignment name** field stays as `Defender Default` / `SecurityCenterBuiltIn` (read-only).
   - **Exemption name:** for example, `Exempt-AzureBackup-ThirdPartyBackup` (append `-Sub`, `-RG-<name>`, or `-VM-<name>` if you create more than one).
   - **Display name:** `Azure Backup recommendation - Third-party backup product`.
   - **Exemption category:** **Mitigated**.
   - **Exemption expiration:** `4/29/2027 12:00 AM UTC`.
   - **Description:** `Workloads protected by approved third-party backup solution. See change ticket CHG0012345.`
4. On the **Policies** tab, select **Azure Backup should be enabled for virtual machines**.
5. **Review + create** > **Create**.

> [!NOTE]
> The exemption is always created **as a child of the scope you pick on the Basics tab**, but it always **targets the assignment you opened** (the subscription-level `SecurityCenterBuiltIn`). That is why opening the assignment from a resource-group blade still lets you exempt only that RG or a single VM under it.

## Mitigated vs Waiver — choosing a category

| Use `Mitigated` when… | Use `Waiver` when… |
|---|---|
| A third-party tool already meets the policy intent (this article's scenario). | The risk is being temporarily accepted while remediation is planned. |
| A different Azure service or on-host configuration provides equivalent control. | The resource is scheduled for decommission. |
| Compliance reporting should reflect that the control is satisfied by another mechanism. | The exemption should clearly be revisited at a future date — pair with `expiresOn`. |

Defender for Cloud's secure score behavior matches Azure Policy:

- **Mitigated** does not award secure score points, but unhealthy resource score penalties are removed, so the effective score increases.
- **Waiver** also stops the affected resources from impacting the score, while signaling accepted risk in compliance reporting.

## Expiration behavior

- `expiresOn` must be ISO 8601 UTC (for example, `2027-04-29T00:00:00Z`).
- After the timestamp passes, the exemption is **no longer honored**, but the exemption resource is **not deleted automatically**.
- Establish a cadence (for example, quarterly) to review exemptions in **Policy** > **Exemptions** and delete or renew any that have expired.

## Parity notes vs. the Defender for Cloud portal experience

- Compliance and Defender for Cloud can take **up to 24 hours** to reflect a new exemption — same as the commercial portal.
- A recommendation that appears in **multiple initiatives** (for example, a regulatory standard you assigned in addition to MCSB) requires **one exemption per assignment**.
- Custom recommendations are not exempted via this flow — manage those at the custom policy assignment directly.

## Verify

1. In the Azure Government portal, navigate to **Policy** > **Exemptions** at the target scope and confirm the exemption is listed.
2. Open **Policy** > **Compliance**, filter by **Compliance state = Exempt**, and confirm the recommendation/resource appears.
3. In **Microsoft Defender for Cloud** > **Recommendations**, open **Azure Backup should be enabled for virtual machines**, switch to the **Not applicable** tab, and confirm the resource appears with **Reason = Exempt**. Allow up to 24 hours.

## Cleanup

1. In the Azure Government portal, navigate to **Policy** > **Exemptions** at the scope where the exemption was created.
2. Select the exemption (for example, `Exempt-AzureBackup-ThirdPartyBackup`).
3. Select **Delete** from the command bar and confirm.
