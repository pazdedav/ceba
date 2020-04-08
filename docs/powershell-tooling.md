# Various PowerShell tools, modules, tips and tricks

## POSH-GIT

POSH-GIT is a PowerShell module for GitHub supplied as part of Git for Windows. When GitHub for Windows is installed, the PowerShell module for GitHub, Git Shell, is also installed, allowing you to clone repositories, send pull requests, among other things, all from the command line.

- Make sure that GitHub for Windows is installed
- Open the Git Shell. You can alternatively use `Import-Module Posh-Git` to import the module, but you’ll need to change the working directory, or specify the full path to the posh-git.psm1 PowerShell module file, to the GitHub for Windows LocalAppData directory.

## Debug option in cmdlets

Example: `Get-AzStorageAccount -ResourceGroupName $rgname -Debug`

