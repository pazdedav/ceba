# Azure Resource Manager Templates Authoring

## Tools in Visual Studio Code

- **Snippets** - example: `arm!` will create a scaffold code for RG-level deployment; snippets for many resource types with required structure
- **ARM Template Outline** - allows you to view the structure in sections
- **Problems** - validation against schemas for resources (e.g. it will highlight missing required properties)
- **Auto-complete** - use Ctrl+Space keyboard shortcut
- **Color-coding** - functions/expressions in yellow, variables in blue, parameters in green
- **Go to Definition** - you can jump from 'Resources' section to Variables or Parameters quickly
- **ARM Params Generator extension** - allows to generate or update an existing `params.json` file (Cmd palette: Azure ARM: Generate parameter file)
- **ARM Template Viewer extension** - a great extension from Ben Colleman (Cmd palette: ARM Viewer: Preview ARM file graphically)
- **Deploy to Azure extension**:
  - auto-generates a CI/CD pipeline (GH Actions or AzDO Pipelines) as a YAML file (2 formats) pre-populated with build and release tasks;
  - deploy app code present in your local system, GH or AzDO repo;
  - workflow also configures AzDO/GH with relevant Azure config + repo-related config (plumbing);
  - Cmd palette: Deploy to Azure: Configure pipeline - in the workflow, the extension will inspect your code and generate a pipeline optimized for your project
  - Current support (March 2020): Node.js app to App Service or Functions, containerized app (Dockerfile required) to AKS
- **What-If feature** - allows to see, what changes will my new deployment bring
  - Syntax: `New-AzDeploymentWhatIf -ScopeType ResourceGroup -ResourceGroupName alex-test -TemplateFile .\what-if-demo.json`
  - project, what you are going to create (resolve vars, functions, ...)
  - no state file (direct pull from ARM API)
  - present diff in a color-coded way
- **ARM DeploymentScripts** - embed scripts (last mile scenarios)
  - new resource type: `deploymentScripts`
  - Example:

  ```json
     {
        "type": "Microsoft.Resources/deploymentScripts",
        "name": "[variables('scriptName')]", // can use newGuid() to save each execution separately
        "apiVersion": "2019-10-01-preview",
        "location": "[parameters('location')]",
        "kind": "AzurePowerShell", // option for 'AzureCLI'
        "identity": {
          "type": "userAssigned",
          "userAssignedIdentities": {
            "[parameters('identity')]": {} // resourceId of the managed instance
            }
          },
        "properties": {
          "azPowerShellVersion": "2.7",
          "primeryScriptUri":
            "https://raw.githubusercontent.com/alex-frankel/VbDemos/master/misc/create-cert.ps1"
          "arguments":
            "-vaultName alex-test-kv -certificateName test-cert -sbjectName 'CN=contoso.com'",
          "timeout": "PT30M",
          "retentionInterval": "PID", // time until deploymentScript will delete itself
          "cleanupPreference": "OnSuccess", // option for "Always", "OnExpiration"
          "forceUpdateTag": "[parameters('timestamp')]" // utcNow()
        }
    },
    "outputs": {
      "results": {
        "type": "string",
        "value": "[reference(variables('scriptName')).outputs.certThumbprint]"
  ```
  - in the background, two resources get created: ACI instance, and SA
  - kind property determines, what container image will be selected
  - systemAssignedIdentity will be supported later
  - cleanupPreference - when ARM will remove ACI and SA
  - New-AzDeployment
  - Dashboard - Deployment Scripts - goves STD_OUT, outputs (requires a special URL: https://ms.portal.azure.com/?feature.showassettypes=Microsoft_Azure_TemplateSpecs...