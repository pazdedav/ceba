# Overview of Resource Manager Template Functions

## Conditions

The ARM template language includes the “condition” key, which can be applied to a resource to determine whether or not that resource is executed.

**Example of condition for Public IP:**

```json
{
    "apiVersion": "2017-04-01",
    "condition": "[equals(parameters('NetworkInterfaceType'),'Public')]",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[Concat(variables('NICName'),'-pip')]",
    "location": "[resourceGroup().location]",
    "tags": {
        "displayName": "[Concat(variables('NICName'),'-pip')]"
    },
    "properties": {
        "publicIPAllocationMethod": "[parameters('IPAllocationMethod')]",
        "dnsSettings": {
            "domainNameLabel": "[Concat(variables('NICName'),'-pip')]"
        }
    }
}
```

- if your condition evaluates to false then the system simply skips, or no-ops the section, it is still in the template
- This is important if you are doing something like new or existing, where subsequent resources will have a dependency on your conditional resource, they do still exist in the template so this won’t error.
- you apply the condition at the resource level, you can’t apply it to sections inside the resource. So what we can’t do is have a single network card resource and choose which child sections are run. What we instead have to do is **create two network card resources and set the condition to control which one actually gets created**. One thing to note is that you can’t have the “name” property be the same for both NICS. Even though you will only create one or the other, the template still needs to compile with both in and it will throw an error if they are named the same.

```json
{
    "apiVersion": "2017-04-01",
    "condition": "[equals(parameters('NetworkInterfaceType'),'Public')]",
    "type": "Microsoft.Network/networkInterfaces",
    "name": "[concat(variables('NICName'),'-public')]",
    "location": "[resourceGroup().location]",
    "tags": {
        "displayName": "parameters('NICName')"
    },
    "dependsOn": [
        "[concat('Microsoft.Network/publicIPAddresses/', variables('NICName'),'-pip')]"
    ],
    "properties": {
        "ipConfigurations": [
            {
                "name": "ipconfig1",
                "properties": {
                    "privateIPAllocationMethod": "[parameters('IPAllocationMethod')]",
                    "subnet": {
                        "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', variables('NetworkName')), '/subnets/',variables('Subnet1Name'))]"
                    },
                    "publicIPAddress": {
                        "id": "[resourceId('Microsoft.Network/publicIPAddresses',Concat(variables('NICName'),'-pip'))]"
                    }
                }
            }
        ]
    }
},
{
    "apiVersion": "2017-04-01",
    "condition": "[equals(parameters('NetworkInterfaceType'),'Private')]",
    "type": "Microsoft.Network/networkInterfaces",
    "name": "[concat(variables('NICName'),'-private')]",
    "location": "[resourceGroup().location]",
    "tags": {
        "displayName": "parameters('NICName')"
    },
    "dependsOn": [],
    "properties": {
        "ipConfigurations": [
            {
                "name": "ipconfig1",
                "properties": {
                    "privateIPAllocationMethod": "[parameters('IPAllocationMethod')]",
                    "subnet": {
                        "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', variables('NetworkName')), '/subnets/',variables('Subnet1Name'))]"
                    }
                }
            }
        ]
    }
}
```

Resource: [link](https://samcogan.com/conditions-in-arm-templates-the-right-way/?utm_content=buffer46b77&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer)

## Cross Resource Group Deployment

You can use **nested deployments** and target different RGs.

```json
{
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "<nestedDeploymentName>",
            "type": "Microsoft.Resources/deployments",
            "resourceGroup": "<crossRGName>",
            "properties": {}
        }
    ]
}
```
