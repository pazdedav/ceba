# ARM Template Deployment Scopes

## Resource Group Scope

Resource deployments to a particular RG

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "LambdaToysRG" -TemplateFile "D:\templates\mainTemplate.json" -TemplateParameterFile "D:\templates\mainTemplate.params.json"
```

```bash
az group deployment create -g "LambdaToysRG" --template-file @mainTemplate.json --parameters @mainTemplate.params.json
```

## Subscription Scope

For resources that exist outside of a resource group, or are applied directly to subscriptions such as:

- Resource Groups
- Policy definitions and assignments
- RBAC Permissions on Subscriptions
- RBAC Custom Roles

```powershell
New-AzSubscriptionDeployment -Location "West Europe" -TemplateFile "D:\templates\mainTemplate.json" -TemplateParameterFile "D:\templates\mainTemplate.params.json"
```

```bash
az deployment create --location WestEurope --template-file @mainTemplate.json --parameters @mainTemplate.params.json
```

Example template [here](https://github.com/Azure/azure-quickstart-templates/tree/master/subscription-level-deployments)

_Note: The commands above do not specify the subscription name or ID, they are using the currently selected subscription in the PowerShell/CLI context._

## Management Group Scope

Define resources at this level:

- Policy definitions and assignments
- RBAC Permissions on Management Groups
- RBAC Custom Roles

```powershell
New-AzManagementGroupDeployment -ManagementGroupId "lambdaToysDevelopmentMgmt" -Location "West Europe" -TemplateFile "D:\templates\mainTemplate.json" -TemplateParameterFile "D:\templates\mainTemplate.params.json"
```

Example template [here](https://github.com/Azure/azure-quickstart-templates/tree/master/managementgroup-level-templates)

_Note: There is currently (March 2020) no CLI equivalent._

## Tenant Scope

Allows you to deploy at the top-level tenant level.

- RBAC Custom Roles
- Policy Definitions

```powershell
New-AzTenantDeployment -Location "West Europe" -TemplateFile "D:\templates\mainTemplate.json" -TemplateParameterFile "D:\templates\mainTemplate.params.json"
```

Example template [here](https://github.com/Azure/azure-quickstart-templates/tree/master/tenant-level-deployments)

_Note: There is currently (March 2020) no CLI equivalent._

## Combining Scopes (multi-scope deployment)

- gives a possibility to deploy resources across scopes (e.g. a policy at the MG level, create RGs in a sub, create resources inside one RG)
- instead of placing deployments in three separate files and call the 3 different PowerShell commands, we can combine these into a single deployment
- this requires **nested templates**
- all of the scopes support another resource, the **deployment resource**. We can use nested templates to create a deployment at another scope.
- select the cmdlet representing the highest scope in your deployment, e.g. `New-AzManagementGroupDeployment`
- the level of nesting can be confusing - instead of using inline templates, use template links to call out to separate template files containing your actual logic for each scope.

### Example of multi-scoped deployment

Example of a MG scoped combined template with inline templates [here](https://github.com/sam-cogan/Demos/blob/master/Template%20Scopes/demo.json)

Example command:

```powershell
New-AzManagementGroupDeployment -ManagementGroupId "lambdaToysDevelopmentMgmt" -Location "West Europe" -TemplateFile "D:\templates\demo.json" -TemplateParameterFile "D:\templates\demo.params.json"
```

_Original article from Sam Cogan [here](https://samcogan.com/arm-template-deployment-scopes/)_
