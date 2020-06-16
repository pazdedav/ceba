# Miscellaneous notes

## Principles and best practices

- One key recommendation from the Twelve-Factor App [guide](https://12factor.net/) is to **separate configuration from code** - e.g. by using Azure App Config

## Azure VM Management

- **Hybrid Benefit** - you might be eligible for Hybrid Benefit for Windows Server and MS SQL Server licenses, if you have Software Assurance for licenses. Recommended approach:
  - Check you [eligibility](https://azure.microsoft.com/en-us/pricing/hybrid-benefit/)
  - Read a [blogpost](http://azurefabric.com/adding-azure-hybrid-benefit-to-the-arm-template-deployment/) explaining some details
  - Include Hybrid Benefit in your "Infrastructure as Code" templates by [adding](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/hybrid-use-benefit-licensing#template) `"licenseType": "Windows_Server"` under VM's properties.

## Windows Package Manager (Preview)

- native package manager for Windows
- [GitHub repo](https://github.com/microsoft/winget-cli)
- install any app with a valid manifest (winget is pre-configured to point to the Microsoft community repository
- commands to help with manifest creation and validation (hash and validate)
- once the first third-party repository is published, you will be able to add that repository as a source as well
- [Documentation page](https://docs.microsoft.com/windows/package-manager)

```powershell
winget search <foo>
winget show <foo>
winget install terminal
```
