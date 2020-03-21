# Azure Blueprints as Code

## Introduction

- Blueprints rely on Az.Blueprint PSH module: `Install-Module -Name Az.Blueprint`
- A blueprint consists of the main blueprint json file and a series of artifact json files
- Create a folder or directory on your computer to store all of your blueprint files. The name of this folder will be the default name of the blueprint unless you specify a new name in the blueprint json file.

  ```
  Blueprint directory (also the default blueprint name)
  * blueprint.json
  * artifacts
      - artifact.json
      - ...
      - more-artifacts.json
  ```
- a variety of expressions and [functions](https://docs.microsoft.com/en-us/azure/governance/blueprints/reference/blueprint-functions) can be used in either a blueprint defintion or artifact such as `concat()` and `parameters()`.

## Blueprint definition

- your main Blueprint file, typically `blueprint.json`
- the blueprint must be created in Azure before any artifacts (policy, role, template). _The artifacts are child resources of a blueprint._ The `Az.Blueprint` module takes care of this for you automatically.

  Example boilerplate blueprint file:
  ```json
  {
      "properties": {
          "description": "This will be displayed in the essentials, so make it good",
          "targetScope": "subscription",
          "parameters": {
              "principalIds": {
                  "type": "string",
                  "metadata": {
                      "displayName": "Display Name for Blueprint parameter",
                      "description": "This is a blueprint parameter that any artifact can reference. We'll display these descriptions for you in the info bubble",
                      "strongType": "PrincipalId"
                  }
              },
              "genericBlueprintParameter": {
                  "type": "string"
              }
          },
          "resourceGroups": {
              "SingleRG": {
                  "description": "An optional description for your RG artifact. FYI location and name properties can be left out and we will assume they are assignment-time parameters",
                  "location": "eastus"
              }
          }
      },
      "type": "Microsoft.Blueprint/blueprints"
  }
  ```

- two optional blueprint parameters - `principalIds` and `genericBlueprintParameter` can be referenced in any artifact
- The `resourceGroups` artifacts are declared here, not in their own files
  - the example hardcodes a location for the resource group of eastus
  - Sets a placeholder name SingleRG for the resource group. The placeholder is just to help you organize the definition and serves as a reference point for your artifacts. Optionally you could hardcode the resource group name by adding "name": "myRgName" as a child property of the SingleRG object.
- Full specification of a [blueprint](https://docs.microsoft.com/en-us/rest/api/blueprints/blueprints/createorupdate#blueprint)

## Artifacts definition

  Example `policyAssignment.json` artifact definition:
  ```json
  {
      "properties": {
          "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/a451c1ef-c6ca-483d-87d-f49761e3ffb5",
          "parameters": {},
          "dependsOn": [],
          "displayName": "My Policy Definition that will be assigned (Currently auditing usage of custome roles)"
      },
      "kind": "policyAssignment",
      "type": "Microsoft.Blueprint/blueprints/artifacts"
  }
  ```

### Common properties

- `Kind` - can be: `template`, `roleAssignment`, `policyAssignment`
- `Type` – this will always be: `Microsoft.Blueprint/blueprints/artifacts`
- `properties` – this is what defines the artifact itself. Some properties of properties are common while others are specific to each type.
- Common properties:
  - dependsOn - optional. You can declare dependencies to other artifacts by referencing the artifact name (which by default is the filename without .json). More info here.
  - resourceGroup – optional. Use the resource group placeholder name to target this artifact to that resource group. If this property isn't specified it will target the subscription.

### Specifications

- Original GitHub repository [here](https://github.com/Azure/azure-blueprints)
- Full boilerplate code [example](https://github.com/Azure/azure-blueprints/tree/master/samples/101-boilerplate)
- Specification of [policy assignment](https://docs.microsoft.com/en-us/rest/api/blueprints/artifacts/createorupdate#policyassignmentartifact)
- Specification of [role assignment](https://docs.microsoft.com/en-us/rest/api/blueprints/artifacts/createorupdate#roleassignmentartifact)
- Specification of [deployment template](https://docs.microsoft.com/en-us/rest/api/blueprints/artifacts/createorupdate#templateartifact)
- Blueprint [samples](https://docs.microsoft.com/en-us/azure/governance/blueprints/samples/?WT.mc_id=social-twitter-stmuraws)

### Parameters

- Nearly everything in a blueprint definition can be parameterized, except `roleDefinitionId` and` policyDefinitionId` in the `rbacAssignment` and `policyAssignment` artifacts respectively.
- Parameters are defined in the main blueprint file and can be referenced in any artifact.
- You can use the same properties you can in an ARM template like `defaultValue`, `allowedValues`

  _Tip: You can also use the `New-AzBlueprintArtifact` cmdlet to convert a standard ARM template into a blueprint artifact, e.g. `New-AzBlueprintArtifact -Type TemplateArtifact -Name storage-account -Blueprint $bp -TemplateFile C:\StorageAccountArmTemplate.json -ResourceGroup "storageRG" -TemplateParameterFile "C:\StorageAccountParams.json"`_

### Passing values between artifacts

- when you need to pass the output from one artifact as the input to another artifact that is deployed later in the blueprint assignment sequence.
- make use of the `artifacts()` function which lets you reference the details of a particular artifact.

  Example - define output in `template` artifact:
  ```json
  {
      "outputs": {
          "storageAccountId": {
              "type": "string",
              "value": "[reference(variables('storageAccountName'), '2016-01-01', 'Full').resourceId]"
          }
      }
  }
  ```

  Example - pass the output as a parameter in another artifact:
  ```json
  {
      "kind": "template",
      "name": "vm-using-storage",
      "properties": {
          "template": {
              ...
          },
          "parameters": {
              "blueprintStorageId": {
                  "value": "[artifacts('storage').outputs.storageAccountId]"
              }
          }
      },
      "type": "Microsoft.Blueprint/blueprints/artifacts"
  }
  ```

### Sequencing order of artifact deployments

- you can use the `dependsOn` property to take a dependency on another artifact.

  Example of `dependsOn` property in an artifact:
  ```json
  {
      "kind": "template",
      "properties": {
        ...
        "dependsOn": ["policyAssignment"],
        ...
      }
  }
  ```

### Publishing the blueprint to Azure

- Example: `Import-AzBlueprintWithArtifact -Name Boilerplate -ManagementGroupId "DevMG" -InputPath  ".\samples\101-boilerplate"`
- Example `build.ps1` pipeline [script](https://github.com/Azure/azure-blueprints/blob/master/pipelines-scripts/build.ps1)
- Example `release.ps1` pipeline [script](https://github.com/Azure/azure-blueprints/blob/master/pipelines-scripts/release.ps1)


## CI/CD - Build phase

- BP definition import together with artifacts (in draft mode) using `Import-AzBlueprintWithArtifact` that looks for `Blueprint.json` file (definition without artefacts).
- Cmdlet will also look for a sub-folder named `Artifacts` in the same input path. All artifacts (template, roleAssignment, policyAssignment) are stored as separate JSON files

  Example `Blueprint.json` file:
  ```json
  {
      "properties": {
          "targetScope": "subscription",
          "parameters": {
              "spnRuntimeIdParameter": {
                  "type": "string"
              }
          },
          "resourceGroups": {
              "apimRG": {
                  "description": "This resource group will be used for API Management"
              }
          }
      },
      "type": "Microsoft.Blueprint/blueprints"
  }
  ```

- BP publishing (turns BP version into R/O mode, can be assigned)
- The `Publish-AzBlueprint` will publish a version of the blueprint that we get using `Get-AzBlueprint`

## CI/CD - Release phase

- assign the latest published blueprint to a subscription using `New-AzBlueprintAssignment` command (requires a path to a JSON file that defines the blueprint that should be published together with parameters)

  Example `Assign.json` file:
  ```json
  {
      "identity": {
          "type": "SystemAssigned"
      },
      "location": "westeurope",
      "properties": {
          "blueprintId": "{{BLUEPRINTID}}",
          "resourceGroups": {
              "apimRG": {
                  "name": "artsbxwe-apim-rg",
                  "location": "westeurope"
              }
          },
          "locks": {
              "mode": "none"
          },
          "parameters": {
              "spnRuntimeIdParameter": {
                  "value": "6d39dfe5-4c33-45f5-841a-9927d6a5e00d"
              }
          }
      }
  }
  ```

## CI/CD using GitHub Actions

- Two jobs are declared : Build and Deploy.
- In the Build job, there are three steps which are: the checkout and the scripts to import and publish the blueprint
- In the Deploy job, there are two steps to checkout and assign the latest published blueprint.

  Example `main.yml` file:
  ```yaml
  name: CI

  on: [push]

  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
      - uses: actions/checkout@v1
      - name: Import the blueprint
        env:
          SUBSCRIPTIONID: fa971420-4388-457a-ac56-1cd453785f14
          BLUEPRINTPATH: ./infrastructure
          BLUEPRINTNAME: testBluePrint
          SPNID: 529a5a8d-4dc4-4f1a-9d4f-3ba5f130446f
          SPNPASS: ${{ secrets.spnPass }}
          TENANTID: eb73c9bc-7f90-4841-9556-c3a56cc82c79
          MGID: charotmg
        run: ./00-importBlueprint.ps1
        shell: pwsh
      - name: Publish the blueprint
        env:
          SUBSCRIPTIONID: fa971420-4388-457a-ac56-1cd453785f14
          BLUEPRINTPATH: ./infrastructure
          BLUEPRINTNAME: testBluePrint
          SPNID: 529a5a8d-4dc4-4f1a-9d4f-3ba5f130446f
          SPNPASS: ${{ secrets.spnPass }}
          TENANTID: eb73c9bc-7f90-4841-9556-c3a56cc82c79
          VERSION: 2.0.0
          MGID: charotmg
        run: ./01-publishBlueprint.ps1
        shell: pwsh
    deploy:
      needs: build
      runs-on: ubuntu-latest
      steps:
      - uses: actions/checkout@v1
      - name: Assign the blueprint
        env:
          SUBSCRIPTIONID: fa971420-4388-457a-ac56-1cd453785f14
          ASSIGNMENTFILE: ./infrastructure/assign.json
          BLUEPRINTNAME: testBluePrint
          SPNID: 529a5a8d-4dc4-4f1a-9d4f-3ba5f130446f
          SPNPASS: ${{ secrets.spnPass }}
          TENANTID: eb73c9bc-7f90-4841-9556-c3a56cc82c79
          VERSION: 1.0.0
        run: ./02-assignBlueprint.ps1
        shell: pwsh
  ```

  Example environment variables:
  ```
  env:
  SUBSCRIPTIONID: fa971420-4388-457a-ac56-1cd453785f14
  BLUEPRINTPATH: ./infrastructure
  BLUEPRINTNAME: testBluePrint
  SPNID: 529a5a8d-4dc4-4f1a-9d4f-3ba5f130446f
  SPNPASS: ${{ secrets.spnPass }}
  TENANTID: eb73c9bc-7f90-4841-9556-c3a56cc82c79
  MGID: charotmg
  ```

  Example import script:
  ```powershell
  param(
      [string]$subscriptionId = $env:SUBSCRIPTIONID,
      [string]$mgId = $env:MGID,
      [string]$blueprintPath = $env:BLUEPRINTPATH,
      [string]$blueprintName = $env:BLUEPRINTNAME,
      [string]$spnId = $env:SPNID,
      [string]$spnPass = $env:SPNPASS,
      [string]$tenantId = $env:TENANTID,
      [string]$version = $env:VERSION
  )

  write-output "Subscription : $subscriptionId"
  $securePass = ConvertTo-SecureString $spnPass -AsPlainText -Force
  $credential = New-Object -TypeName pscredential -ArgumentList $spnId, $securePass
  Login-AzAccount -Credential $credential -ServicePrincipal -TenantId $tenantId

  $createdBlueprint = Get-AzBlueprint -ManagementGroupId $mgId -Name $blueprintName -errorAction SilentlyContinue

  if($createdBlueprint)
  {
      Publish-AzBlueprint -Blueprint $createdBlueprint -Version $version
  }else
  {
      throw "Could not get Blueprint"
      exit 1
  }
  ```

  Example publish script:
  ```powershell
  param(
      [string]$subscriptionId = $env:SUBSCRIPTIONID,
      [string]$mgId = $env:MGID,
      [string]$blueprintPath = $env:BLUEPRINTPATH,
      [string]$blueprintName = $env:BLUEPRINTNAME,
      [string]$spnId = $env:SPNID,
      [string]$spnPass = $env:SPNPASS,
      [string]$tenantId = $env:TENANTID,
      [string]$version = $env:VERSION
  )

  write-output "Subscription : $subscriptionId"
  $securePass = ConvertTo-SecureString $spnPass -AsPlainText -Force
  $credential = New-Object -TypeName pscredential -ArgumentList $spnId, $securePass
  Login-AzAccount -Credential $credential -ServicePrincipal -TenantId $tenantId

  $createdBlueprint = Get-AzBlueprint -ManagementGroupId $mgId -Name $blueprintName -errorAction SilentlyContinue

  if($createdBlueprint)
  {
      Publish-AzBlueprint -Blueprint $createdBlueprint -Version $version
  }else
  {
      throw "Could not get Blueprint"
      exit 1
  }
  ```

  Example assignment script:
  ```powershell
  param(
      [string]$subscriptionId = $env:SUBSCRIPTIONID,
      [string]$assignmentFile = $env:ASSIGNMENTFILE,
      [string]$blueprintName = $env:BLUEPRINTNAME,
      [string]$spnId = $env:SPNID,
      [string]$spnPass = $env:SPNPASS,
      [string]$tenantId = $env:TENANTID,
      [string]$version = $env:VERSION
  )

  Install-Module -Name Az.Blueprint -AllowClobber -Force

  if (!(Get-Module -ListAvailable -Name Az.Blueprint)) {
      throw "Module does not exist"
      exit 1
  }

  $securePass = ConvertTo-SecureString $spnPass -AsPlainText -Force
  $credential = New-Object -TypeName pscredential -ArgumentList $spnId, $securePass
  Login-AzAccount -Credential $credential -ServicePrincipal -TenantId $tenantId

  $createdBlueprint = Get-AzBlueprint -SubscriptionId $subscriptionId -Name $blueprintName -LatestPublished -errorAction SilentlyContinue

  if($createdBlueprint)
  {
          (Get-Content $assignmentFile).replace("{{BLUEPRINTID}}",$createdBlueprint.id) | Set-Content $assignmentFile
          New-AzBlueprintAssignment -Name "assigned-$blueprintName" -Blueprint $createdBlueprint -AssignmentFile $assignmentFile -SubscriptionId $subscriptionId
  }else
  {
      throw "Could not get Blueprint"
      exit 1
  }
  ```

  Import script [source](https://github.com/charotAmine/BlueprintsAsCode/blob/master/00-importBlueprint.ps1)
  Publish script [source](https://github.com/charotAmine/BlueprintsAsCode/blob/master/01-publishBlueprint.ps1)
  Assignment script [source](https://github.com/charotAmine/BlueprintsAsCode/blob/master/02-assignBlueprint.ps1)


Important:

- secrets (such as the SPN Password/key) should be stored in GitHub Secrets in the repo
- You can access to it using `${{ secrets.SECRETNAME }}`

## Additional documentation

- GitHub [repository](https://github.com/Azure/azure-blueprints)
- Original article from Amine Charot [here](https://medium.com/@charotamine/azure-blueprints-as-code-github-actions-c0331152ded8)
