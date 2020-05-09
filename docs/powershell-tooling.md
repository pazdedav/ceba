# Various PowerShell tools, modules, tips and tricks

## POSH-GIT

POSH-GIT is a PowerShell module for GitHub supplied as part of Git for Windows. When GitHub for Windows is installed, the PowerShell module for GitHub, Git Shell, is also installed, allowing you to clone repositories, send pull requests, among other things, all from the command line.

- Make sure that GitHub for Windows is installed
- Open the Git Shell. You can alternatively use `Import-Module Posh-Git` to import the module, but you’ll need to change the working directory, or specify the full path to the posh-git.psm1 PowerShell module file, to the GitHub for Windows LocalAppData directory.

## Debug option in cmdlets

Example: `Get-AzStorageAccount -ResourceGroupName $rgname -Debug`

## Connecting to multiple Azure environments using context

To hold credential information, like user and subscription. PowerShell uses context objects. By using `AzContext` comandlets, you can have multiple Powershell Azure contexts available in the same PowerShell session. This allows for easy switching between multiple environments and profiles, including different tenants.

```powershell
# Connect to Azure specifying a tenant
# If you want to connect to multiple tenants, you can connect multiple times.
Connect-AzAccount -tenantId customer1.onmicrosoft.com

# adding a new PowerShell Azure context
# setting a friendly name to allow for easy switching.

Set-AzContext -name "Subscription 1 in tenant 1" -SubscriptionId "31ffbc99-4cbf-43b2-8789-ba8d73171e70" -tenantid customer1.onmicrosoft.com
Set-AzContext -name "Subscription 2 in tenant 1" -SubscriptionId "b5c85827-0afd-49a0-8923-8fe35cfa8dd0" -tenantid customer1.onmicrosoft.com

# check the current context
Get-AzContext

# Get all contexts available
Get-AzContext -ListAvailable

#Switch between the contexts available
Select-AzContext 'Subscription 1 in tenant 1'

#Clear all contexts/log out
Clear-AzContext -force
```

- Original [source](https://adatum.no/powershell/multiple-azure-credentials-in-powershell)
