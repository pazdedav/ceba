# Azure Blueprints as Code

## Build phase

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

## Release phase

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
