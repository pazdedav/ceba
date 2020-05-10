# Design Policy as Code Workflows

## Introduction

- PaC practice allows a shift from manually managing each policy definition in the Azure portal to something more manageable and repeatable at enterprise scale.
- PaC is the combination of two predominant approaches to managing systems at scale in the cloud: Infrastructure as Code and DevOps
- Allows you to keep your policy definitions in source control and whenever a change is made, test and validate that change. The validation step should also be a component of other continuous integration or continuous deployment workflows. Examples include deploying an application environment or virtual infrastructure. By making Azure Policy validation an early component of the build and deployment process the application and operations teams discover if their changes are non-complaint, long before it's too late and they're attempting to deploy in production.

## Workflow overview

![workflow-diagram](https://docs.microsoft.com/en-us/azure/governance/policy/media/policy-as-code/policy-as-code-workflow.png)

## Create and update policy definitions

- The policy definitions are created using JSON, and stored in source control.
- Each policy has it's own set of files, such as the parameters, rules, and environment parameters, that should be stored in the same folder.
- When a new policy is added or an existing one updated, the workflow should automatically update the policy definition in Azure.
- Example structure:

    ```text
    .
    |
    |- policies/  ________________________ # Root folder for policies
    |  |- policy1/  ______________________ # Subfolder for a policy
    |     |- policy.json _________________ # Policy definition
    |     |- policy.parameters.json ______ # Policy definition of parameters
    |     |- policy.rules.json ___________ # Policy rule
    |     |- params.dev.json _____________ # Parameters for a Dev environment
    |     |- params.prd.json _____________ # Parameters for a Prod environment
    |     |- params.tst.json _____________ # Parameters for a Test environment
    |
    |  |- policy2/  ______________________ # Subfolder for a policy
    |     |- policy.json _________________ # Policy definition
    |     |- policy.parameters.json ______ # Policy definition of parameters
    |     |- policy.rules.json ___________ # Policy rule
    |     |- params.dev.json _____________ # Parameters for a Dev environment
    |     |- params.prd.json _____________ # Parameters for a Prod environment
    |     |- params.tst.json _____________ # Parameters for a Test environment
    |
    ```

### Prerequisites

- Install [ARMClient](https://github.com/projectkudu/ARMClient)
- Update Azure PowerShell and Azure CLI to the latest version
- Install VS Code with recommended extensions (including "Azure Policy")
- Register PolicyInsights RP: `Register-AzResourceProvider -ProviderNamespace 'Microsoft.PolicyInsights'`

### Create and assign a policy definition via PSH

Example definition:

```json
{
    "if": {
        "allOf": [{
                "field": "type",
                "equals": "Microsoft.Storage/storageAccounts"
            },
            {
                "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                "equals": "Allow"
            }
        ]
    },
    "then": {
        "effect": "audit"
    }
}
```

PowerShell command for adding new definition (the last parameter could be replaced with -SubscriptionId):

```powershell
New-AzPolicyDefinition -Name 'AuditStorageAccounts' -DisplayName 'Audit Storage Accounts Open to Public Networks' -Policy 'AuditStorageAccounts.json' -ManagementGroupName ''
```

PowerShell command for adding new assignment

```powershell
$rg = Get-AzResourceGroup -Name 'ContosoRG'
$Policy = Get-AzPolicyDefinition -Name 'AuditStorageAccounts'
New-AzPolicyAssignment -Name 'AuditStorageAccounts' -PolicyDefinition $Policy -Scope $rg.ResourceId
```

## Create and update initiative definitions

- Initiatives have their own JSON file and related files that should be stored in the same folder.
- The initiative definition requires the policy definition to already exist, so can't be created or updated until the source for the policy has been updated in source control and then updated in Azure
- When adding or updating an existing initiative, the workflow should automatically update the initiative definition in Azure.
- Example structure:

    ```txt
    .
    |
    |- initiatives/ ______________________ # Root folder for initiatives
    |  |- init1/ _________________________ # Subfolder for an initiative
    |     |- policyset.json ______________ # Initiative definition
    |     |- policyset.definitions.json __ # Initiative list of policies
    |     |- policyset.parameters.json ___ # Initiative definition of parameters
    |     |- params.dev.json _____________ # Parameters for a Dev environment
    |     |- params.prd.json _____________ # Parameters for a Prod environment
    |     |- params.tst.json _____________ # Parameters for a Test environment
    |
    |  |- init2/ _________________________ # Subfolder for an initiative
    |     |- policyset.json ______________ # Initiative definition
    |     |- policyset.definitions.json __ # Initiative list of policies
    |     |- policyset.parameters.json ___ # Initiative definition of parameters
    |     |- params.dev.json _____________ # Parameters for a Dev environment
    |     |- params.prd.json _____________ # Parameters for a Prod environment
    |     |- params.tst.json _____________ # Parameters for a Test environment
    |
    ```

## Test and validate the updated definition

- Once automation has taken your newly created or updated policy or initiative definitions and made the update to the object in Azure, test the changes that were made by assigning it to Dev resources (platform QA environment).
- The assignment should use `enforcementMode` of `disabled` so that resource creation and updates aren't blocked, but that existing resources are still audited for compliance to the updated policy definition. Even with enforcementMode, it's recommended that the assignment scope is either a resource group or a subscription that is specifically used for validating policies.
- Full testing of policy definition must be done with  PUT and PATCH REST API calls, compliant and non-compliant resources, and edge cases like a property missing from the resource.
- After the assignment is deployed, use the Policy SDK to get compliance data for the new assignment. The environment used to test the policies and assignments should have both compliant and non-compliant resources. Like a good unit test for code, **you want to test that resources are as expected and that you also have no false-positives or false-negatives**. If you test and validate only for what you expect, there may be unexpected and unidentified impact from the policy.

## Enable remediation tasks

- The next step is to validate remediation. Policies that use either `deployIfNotExist`s or `modify` may be turned into a remediation task and correct resources from a non-compliant state.
- Grant the policy assignment the role assignment defined in the policy definition. This role assignment gives the policy assignment managed identity enough rights to make the needed changes to make the resource compliant.
- Once the policy assignment has appropriate rights, use the Policy SDK to trigger a remediation task against a set of resources that are known to be non-compliant.
- Three tests should be completed against these remediated tasks before proceeding:
  - **Validate that the remediation task completed successfully** - Run policy evaluation to see that policy compliance results are updated as expected
  - **Run an environment unit test against the resources directly** to validate their properties have changed
  - **Testing both the updated policy evaluation results and the environment directly** provide confirmation that the remediation tasks changed what was expected and that the policy definition saw the compliance change as expected.

## Update to enforced assignments

After all validation gates have completed, update the assignment to use `enforcementMode` of `enabled`. This change should initially be made in the same environment far from production. Once that environment is validated as working as expected, the change should then be scoped to include the next environment and so on until the policy is deployed to production resources.

## Process integrated evaluations

- Policy evaluation should be part of the deployment process for any workflow that deploys or creates resources in Azure (such as deploying applications or running Resource Manager templates to create infrastructure)
- After the application or infrastructure deployment is done to a test subscription or resource group, policy evaluation should be done for that scope checking validation of all existing policies and initiatives. While they may be configured as enforcementMode disabled in such an environment, it's useful to know early if an application or infrastructure deployment is in violation of policy definitions early.
- This policy evaluation should therefore be a step in those workflows, and fail deployments that create non-compliant resources.
