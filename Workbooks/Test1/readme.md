# Publishing an Azure managed application definition

## Option 1 - Deploy using Azure Portal
### Step 1 - Get authorization info
Run these commands in the Azure Cloud shell
```bash
> userid=$(az ad user show --upn-or-object-id example@contoso.org --query objectId --output tsv)
> roleid=$(az role definition list --name Owner --query [].name --output tsv)
> echo [{\"principalId\":\"$userid\", \"roleDefinitionId\":\"$roleid\" }]
```

Copy the value of returned by the last command to use in the `authorizations` parameter of the next step.

### Step 2 - Deploy
Clicking on the button below, will create the Managed Application definition to a Resource Group in your Azure subscription.

[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Facearun%2Fmanagedsolutions%2Fmaster%2FWorkbooks%2FTest1%2Fazuredeploy.json)

## Option 2 - Deploy Using the Azure Cloud Shell
### Step 1 - Open Cloud Shell
Select the Cloud Shell button on the menu on the upper-right corner of the [Azure portal](https://portal.azure.com).

![Image showing the cloud shell ](https://docs.microsoft.com/en-us/azure/includes/media/cloud-shell-try-it/cloud-shell-menu.png)

### Step 2 - Create a resource group for definition

```bash
az group create --name appDefinitionGroup --location westcentralus
```

### Step 3 - Get User principal
```bash
userid=$(az ad user show --upn-or-object-id example@contoso.org --query objectId --output tsv)
```
### Step 4 - Get Role Id
```bash
roleid=$(az role definition list --name Owner --query [].name --output tsv)
```

### Step 4 - Create the managed app definition
```bash
az managedapp definition create \
  --name "ManagedWorkbooks" \
  --location "westcentralus" \
  --resource-group appDefinitionGroup \
  --lock-level ReadOnly \
  --display-name "Managed Workbooks" \
  --description "Managed Azure Workbooks" \
  --authorizations "$userid:$roleid" \
  --package-file-uri "http://raw.githubusercontent.com/acearun/managedsolutions/master/Workbooks/Test1/test1.zip"
  ```


## Deploying the managed app
Either way you choose to deploy the managed app definition, the next step usually is to create a deployment of the app:
1. Use the deploy option in the Managed App definition.
2. Choose the sub and resource group in the deployment workflow
3. Deploy

## Test1

This template:
1. Deploys two workbooks to the locked resource group. They currently point to no resources - just a static string indicating it is a template. One of the workbooks is supposed to be Spanish while the other is invariant.
2. Deploys the managed app in the selected resource group. The managed group has template metadata including localization information. This information can be found in the `Parameters and Outputs` section of the Managed App under the output `templateMetadata`. 

## Template Metadata
```json
{
    "templates": [
        {
            "source": "arm",
            "id": "[resourceId( 'microsoft.insights/workbooks', parameters('SimpleTemplateEn'))]",
            "galleries": [
                {
                    "name": "Simple Template",
                    "category": "Failures",
                    "type": "tsg",
                    "resourceType": "microsoft.insights/components",
                    "order": 100
                }
            ],
            "localized": {
                "es-es": {
                    "id": "[resourceId( 'microsoft.insights/workbooks', parameters('SimpleTemplateEs'))]",
                    "galleries": [
                        {
                            "name": "Sencillo Template",
                            "category": "Fallos",
                            "type": "tsg",
                            "resourceType": "microsoft.insights/components",
                            "order": 100
                        }
                    ]
                }
            }
        }
    ]
}
```


