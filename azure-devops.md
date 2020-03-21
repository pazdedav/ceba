# Azure DevOps

## Working with AzDO CLI

Azure DevOps extension for Azure Command Line Interface (CLI)

- allows to work with Pipelines, Boards, Repos, Artifacts and DevOps commands
- Steps:
  - Install the Azure CLI - [download](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
  - Add the Azure DevOps Extension: `az extension add --name azure-devops`
  - Run the az login command to authenticate user: `az login`
  - configure the default project and organization: `az devops configure --defaults organization=https://dev.azure.com/{{OrganizationName}} project={{ProjectName}}`
  - create a repo: `az repos create --name "CLISample" --detect true --open --organization "https://dev.azure.com/sausha" --project "PublicData"`
  - create a pipeline: `az pipelines create --name "MyCLISamplee" --description "Pipeline for CLI project" --repository CLISample --branch master --repository-type tfsgit` _(tfsgit' for Azure Repos, 'github' for GitHub)_

**Resources and info:

- Azure DevOps CLI Extension GitHub repo - [link](https://github.com/Azure/azure-devops-cli-extension)
- Azure DevOps CLI documentation - [link](https://docs.microsoft.com/azure/devops/cli/get-started?view=azure-devops?WT.mc_id=devopslab-c9-dabrady)
- az devops -h
