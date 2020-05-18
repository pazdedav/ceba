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

## MSINFO32-like cmdlet

`Get-ComputerInfo`

- Want to know if the device is joined to AD?  Check CsPartOfDomain.  Want to know what domain?  Look at CsDomain (which will be WORKGROUP on an AAD-joined device).
- Need to know if the device is UEFI or not?  Check to see if the BiosFirmwareType is Uefi.
- Want to know how long Windows has been running since the last reboot?  Check OsUptime or OsLastBootUpTime.
- How about the Windows SKU?  OsOperatingSystemSKU will tell you that.
- Need to get a list of languages that are installed?  OsMuiLanguages has that list.
- [Source](https://oofhours.com/2020/05/13/the-most-useful-powershell-cmdlet-i-didnt-know-existed/)

## Managing AzDO using VSTeam PSH package

- Source: [PowerShell Gallery](https://www.powershellgallery.com/packages/VSTeam/6.4.8)
- TIP: Use `Invoke-VSTeamRequest` (`ivr` alias) cmdlet and use tab complete the areas and resources of every API in AzureDevOps
- Project [site](https://github.com/DarqueWarrior/vsteam)