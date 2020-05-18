# Miscellaneous notes

## Principles and best practices

- One key recommendation from the Twelve-Factor App [guide](https://12factor.net/) is to **separate configuration from code** - e.g. by using Azure App Config

## Azure VM Management

- **Hybrid Benefit** - you might be eligible for Hybrid Benefit for Windows Server and MS SQL Server licenses, if you have Software Assurance for licenses. Recommended approach:
  - Check you [eligibility](https://azure.microsoft.com/en-us/pricing/hybrid-benefit/)
  - Read a [blogpost](http://azurefabric.com/adding-azure-hybrid-benefit-to-the-arm-template-deployment/) explaining some details
  - Include Hybrid Benefit in your "Infrastructure as Code" templates by [adding](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/hybrid-use-benefit-licensing#template) `"licenseType": "Windows_Server"` under VM's properties.
