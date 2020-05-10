# Visual Studio Code Productivity Tips and Tricks

## Azure Policy extension

- look up aliases and review resources and policies
- Changes made locally to policies viewed in the Azure Policy extension for Visual Studio Code aren't synced to Azure.
- Extension name: Azure Policy
- Configure VS Code: File\Preferences\Settings - Azure:Cloud (change if working with national clouds)
- Sign in to Azure: Use this command from the Command Palette: `Azure: Sign In` and `Azure: Select Subscriptions`
- The Azure Policy extension lists resources in the selected subscriptions by Resource Provider and by resource group in the Resources pane.
- By default, the extension filters the 'Resource Provider' portion by existing resources and resources that have policy aliases
- You can search for specific resources via the Command Palette: `Resources: Search Resources`
- When a resource is selected, whether through the search interface or by selecting it in the treeview, the Azure Policy extension opens the JSON file representing that resource and all its Resource Manager property values. Once a resource is open, **hovering over the Resource Manager property name or value displays the Azure Policy alias** if one exists.
- The Azure Policy extension lists policy types and policy assignments as a treeview for the subscriptions selected to be displayed in the Policies pane.
- You can search for specific policies via the Command Palette: `Policies: Search Policies`
