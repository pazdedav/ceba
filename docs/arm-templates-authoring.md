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
