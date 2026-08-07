# Exempt Microsoft Defender for Cloud recommendations in Azure Government using Azure Policy

**Last reviewed:** August 7, 2026

In Azure Government, recommendation exemption rules in Microsoft Defender for Cloud (MDC) are **not available**. For a policy-based recommendation, you can create an **Azure Policy exemption** against the underlying policy assignment, such as the Microsoft Cloud Security Benchmark (MCSB) initiative. This produces an Azure Policy **Exempt** compliance state and can cause the affected resource to appear as **Not applicable** with the reason **Exempt** in Defender for Cloud after compliance re-evaluates.

> [!IMPORTANT]
> This is a best-effort workaround, not full parity with an exemption created through Defender for Cloud. Microsoft recommends creating recommendation exemptions through the Defender for Cloud portal where that capability is available. Exemptions created directly through Azure Policy or its API might not fully integrate with Defender for Cloud or propagate across every assigned initiative. Verify the result in both Azure Policy and Defender for Cloud.

This article uses the recommendation **"Azure Backup should be enabled for virtual machines"** (built-in policy definition `013e242c-8828-4970-87b3-ab247555486d`, member of MCSB) and the scenario *"another (third-party) backup solution already protects these virtual machines"* to demonstrate creating exemptions at the **subscription**, **resource group**, and **individual resource** scopes.

This procedure applies only to recommendations that have an underlying Azure Policy definition. It does not apply to KQL-based recommendations, native Defender for Cloud recommendations without a policy definition, or other recommendations that do not support exemptions.

## Cited resources

- [Cloud feature availability for US Government customers](https://learn.microsoft.com/azure/security/fundamentals/feature-availability)
- [Azure Policy exemption structure](https://learn.microsoft.com/azure/governance/policy/concepts/exemption-structure)
- [Exempt resources from Defender for Cloud recommendations](https://learn.microsoft.com/azure/defender-for-cloud/exempt-resource)
- [Review and manage Defender for Cloud exemptions](https://learn.microsoft.com/azure/defender-for-cloud/review-exemptions)
- [Microsoft Cloud Security Benchmark (MCSB) initiative](https://learn.microsoft.com/security/benchmark/azure/introduction)
- [Built-in policy: Azure Backup should be enabled for virtual machines](https://learn.microsoft.com/azure/governance/policy/samples/built-in-policies#backup)
- [Resource Policy Contributor built-in role](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#resource-policy-contributor)
- [Security Admin built-in role](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#security-admin)

## Exemption fields represented in Azure Policy

| Defender for Cloud (commercial) field | Description |
| --- | --- |
| Scope | Management group, subscription, or selected resources |
| Name | Friendly name for the exemption |
| Expiration date | Optional |
| Category | **Resolved through 3rd party (mitigated)** or **Risk accepted (waiver)** |
| Description / justification | Free text |

Azure Policy represents these fields with two named categories: `Mitigated` and `Waiver`. See [Mitigated vs Waiver — choosing a category](#mitigated-vs-waiver--choosing-a-category) below for guidance.

## Field-to-property mapping

The portal fields above map to the underlying Azure Policy exemption properties (`name`, `displayName`, `expiresOn`, `exemptionCategory`, `policyDefinitionReferenceId`, `description`, `metadata`, `resourceSelectors`) documented in [Azure Policy exemption structure](https://learn.microsoft.com/azure/governance/policy/concepts/exemption-structure). No JSON authoring is required for this walkthrough.

## Prerequisites

### Confirm where the MCSB initiative is assigned

When Microsoft Defender for Cloud is enabled on a subscription, it automatically creates a policy assignment named **`SecurityCenterBuiltIn`** **at the subscription level**. The MCSB initiative is **not** auto-assigned at a management group, resource group, or individual resource.

> [!NOTE]
> The display name shown in the portal for this assignment has changed over time. Depending on tenant, region, and when the assignment was created, you may see any of the following — they all refer to the same underlying assignment (`SecurityCenterBuiltIn`):
>
> - **ASC Default (subscription: \<id\>)** — original display name.
> - **Defender Default** — newer display name (commonly seen in commercial).
> - **Microsoft cloud security benchmark** — current display name in many tenants.

When following the steps below, match on the assignment name `SecurityCenterBuiltIn` rather than the display name.

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

To create an Azure Policy exemption, you need permission to write the exemption at the exemption scope and the `Microsoft.Authorization/policyAssignments/exempt/action` permission on the policy assignment being exempted. The built-in roles below grant these permissions when assigned at a scope that covers both resources.

**Required Azure RBAC role at the exemption scope** (any one of):

- **Owner**
- **Resource Policy Contributor**
- **Security Admin**

The role must be assigned **at the same scope as the exemption (or higher)**:

- **Management group exemption** → role on the management group (or tenant root).
- **Subscription exemption** → role on the subscription (or parent MG).
- **Resource group exemption** → role on the resource group (or parent subscription / MG).
- **Single-resource exemption** → role on that resource (or any parent).

The role assignment must also cover the target policy assignment. For example, a role assigned only on an individual VM can permit exemption-resource operations on that VM but does not cover a subscription-level `SecurityCenterBuiltIn` assignment. Assign the role at the subscription or another common parent scope, or use custom roles at the two required scopes.

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
| --- | --- |
| Recommendation | Azure Backup should be enabled for virtual machines |
| Built-in policy definition ID | `013e242c-8828-4970-87b3-ab247555486d` |
| Initiative | Microsoft Cloud Security Benchmark (`1f3afdf9-d0c9-4c3d-847f-89da613e70a8`) |
| Category | `Mitigated` (third-party backup product is in use) |
| Expiration | Example only: `2027-08-07T00:00:00Z` (one year from this article's review date) |
| Justification | "Workloads protected by approved third-party backup solution. See change ticket CHG0012345." |

## Create the exemption (portal)

The same flow works for subscription, resource group, and single-resource scopes — only the **Exemption scope** value on the **Basics** tab differs.

### Permissions

- Owner / Resource Policy Contributor / Security Admin at a scope that covers both the exemption and the target policy assignment. For the default subscription-level `SecurityCenterBuiltIn` assignment, assign the role at the subscription or a parent management group.

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
   - **Exemption expiration:** choose a date that follows your organization's exception-review policy. For this example, use `8/7/2027 12:00 AM UTC`.
   - **Description:** `Workloads protected by approved third-party backup solution. See change ticket CHG0012345.`
4. On the **Policies** tab, select **Azure Backup should be enabled for virtual machines**.
5. **Review + create** > **Create**.

> [!NOTE]
> The exemption is always created **as a child of the scope you pick on the Basics tab**, but it always **targets the assignment you opened** (the subscription-level `SecurityCenterBuiltIn`). That is why opening the assignment from a resource-group blade still lets you exempt only that RG or a single VM under it.

## Mitigated vs Waiver — choosing a category

| Use `Mitigated` when… | Use `Waiver` when… |
| --- | --- |
| A third-party tool already meets the policy intent (this article's scenario). | The risk is being temporarily accepted while remediation is planned. |
| A different Azure service or on-host configuration provides equivalent control. | The resource is scheduled for decommission. |
| Compliance reporting should reflect that the control is satisfied by another mechanism. | The exemption should clearly be revisited at a future date — pair with `expiresOn`. |

Defender for Cloud's secure score behavior matches Azure Policy:

- **Mitigated** treats the resource as healthy, so the secure score increases.
- **Waiver** excludes the resource from the secure score calculation, resulting in a neutral score effect while recording the accepted risk.

## Expiration behavior

- `expiresOn` must be ISO 8601 UTC (for example, `2027-08-07T00:00:00Z`). Treat dates in this article as examples and select an expiration that follows your organization's governance policy.
- After the timestamp passes, the exemption is **no longer honored**, but the exemption resource is **not deleted automatically**.
- Establish a cadence (for example, quarterly) to review exemptions in **Policy** > **Exemptions** and delete or renew any that have expired.

## Parity notes vs. the Defender for Cloud portal experience

- Compliance and Defender for Cloud can take **up to 24 hours** to reflect a new exemption.
- A recommendation that appears in **multiple initiatives** (for example, a regulatory standard assigned in addition to MCSB) requires **one exemption per applicable policy assignment**. A new initiative assignment does not inherit exemptions created for an existing assignment.
- This flow requires an underlying Azure Policy definition and assignment. KQL-based recommendations, native Defender for Cloud recommendations without a policy definition, and unsupported recommendations cannot use it. A custom policy assignment can have an Azure Policy exemption, but that does not guarantee a corresponding Defender for Cloud recommendation state.

## Verify

1. In the Azure Government portal, navigate to **Policy** > **Exemptions** at the target scope and confirm the exemption is listed.
2. Open **Policy** > **Compliance**, filter by **Compliance state = Exempt**, and confirm the recommendation/resource appears.
3. In **Microsoft Defender for Cloud** > **Recommendations**, open **Azure Backup should be enabled for virtual machines**, switch to the **Not applicable** tab, and confirm the resource appears with **Reason = Exempt**. Allow up to 24 hours.

If the resource remains unhealthy in Defender for Cloud after 24 hours:

1. Confirm that the exemption is active and not expired.
2. Confirm that its scope includes the affected resource.
3. Confirm that `policyDefinitionReferenceId` identifies the intended policy within the initiative assignment.
4. Check whether the recommendation is enabled by another initiative and create a separate exemption for each applicable assignment.
5. Confirm that the recommendation is policy-based and supports exemption.

If Azure Policy reports **Exempt** but Defender for Cloud still reports the resource as unhealthy after these checks, treat the Azure Policy exemption as not integrated with that Defender for Cloud recommendation. This is a documented limitation of exemptions created outside Defender for Cloud.

## Cleanup

1. In the Azure Government portal, navigate to **Policy** > **Exemptions** at the scope where the exemption was created.
2. Select the exemption (for example, `Exempt-AzureBackup-ThirdPartyBackup`).
3. Select **Delete** from the command bar and confirm.
