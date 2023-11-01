---
reviewed: 2023-09-19
---

#azure

https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/overview

## Definition

- An **Azure Resource Manager template** precisely defines all the Resource Manager resources in a deployment. You can deploy a Resource Manager template into a resource group as a single operation.
- Templates follow a declarative syntax.
- Templates are idempotent. Running them multiple times will only cause the changed resources to be updated

## Structure

Templates can be written in JSON or Azure Bicep
Below is the JSON structure

```json
{
	"$schema": "http://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
	"contentVersion": "",
	"parameters": {},
	"variables": {},
	"functions": [],
	"resources": [],
	"outputs": {}
}
```

## Creating custom templates

- When creating a resource from the azure portal, you can have access to its template. Alternatively, go to the resource group then `Deployments` to see all the deployments
- When accessing the resource directly you can click on `Automation|Export template` to generate a template
- Then you can download it to get the template.json and parameters.json file.

- Then in the portal search for `Deploy custom templates` and upload your own file

- To develop templates in VSCode, download Azure Resource Manager (ARM) Tools extension, then type `arm!` in a json file

Example template:

```json
{
	"$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
	"contentVersion": "1.0.0.0",
	"parameters": {
		"storageAccountType": {
			"type": "string",
			"defaultValue": "Standard_LRS",
			"allowedValues": [
				"Standard_LRS",
				"Standard_GRS",
				"Standard_ZRS",
				"Premium_LRS"
			],
			"metadata": {
				"description": "Storage Account type"
			}
		}
	},
	"functions": [],
	"variables": {},
	"resources": [
		{
			"name": "storageaccount1fgb",
			"type": "Microsoft.Storage/storageAccounts",
			"apiVersion": "2019-06-01",
			"tags": {
				"displayName": "storageaccount1fgb"
			},
			"location": "[resourceGroup().location]",
			"kind": "StorageV2",
			"sku": {
				"name": "[parameters('storageAccountType')]",
				"tier": "Premium"
			}
		}
	],
	"outputs": {}
}
```

Deploy it with

```
az deployment group create \
  --name testdeployment1 \
  --template-file $templateFile \
  --parameters storageAccountType=Standard_LRS
```
