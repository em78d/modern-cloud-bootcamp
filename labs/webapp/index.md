# Web Apps in Azure


## Create Web App

#### Open the Azure Portal

1. In an open browser window, navigate to the **Azure Portal** (<https://portal.azure.com>).

#### Open Cloud Shell

1. At the top of the portal, click the **Cloud Shell** icon to open a new shell instance.

#### Create an App Service plan

1. At the **Cloud Shell** command prompt at the bottom of the portal, type in the following command and press **Enter** to create a variable which value designates the name of the resource group you will use in this exercise:

```sh
RESOURCE_GROUP_APP='serverless-RG'
```

1. At the **Cloud Shell** command prompt at the bottom of the portal, type in the following command and press **Enter** to create a variable which value designates the region you will use in this exercise:
```sh
LOCATION='westus2'
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create the resource group:

```sh
az group create --name $RESOURCE_GROUP_APP --location $LOCATION
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new App Service plan:

```sh
az appservice plan create --is-linux --name "serverless-$LOCATION" --resource-group $RESOURCE_GROUP_APP --location $LOCATION --sku F1
```

   
#### Task 4: Create a Web App instance

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to view a list of possible runtimes for a Linux-based App Service web app instance: 

```sh
az webapp list-runtimes --linux --output tsv
``` 

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new variable which value is a randomly generated string that you will use as the name of a new web app:

```sh
WEBAPPNAME1=webapp05021$RANDOM$RANDOM
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new web app using a unique name:

```sh
az webapp create --name $WEBAPPNAME1 --plan advazure-serverless-$LOCATION --resource-group $RESOURCE_GROUP_APP --runtime "DOTNETCORE|2.1"
```

> **Note**: In case the command fails due to duplicate web app name, re-run the last two steps until the command completes successfully  

1. Wait for the deployment to complete before you proceed to the next task.

#### View deployment results

1. In the Azure portal, click **Resource groups**.

1. On the **Resource groups** blade, click **serverless-RG**.

1. On the **serverless-RG** blade, click the entry representing the Azure web app you created earlier in this exercise.

1. On the web app blade, click the **Browse** button at the top of the blade.

1. Review the default page generated by Azure App Service.

1. Close the new browser tab and return to the browser tab displaying the Azure portal.

> **Review**: In this exercise, you created a Linux-based App Service Plan that contained a blank web app.   


## Deploy Web App code

#### Deploy code with a Web App Extension using an Azure Resource Manager template and GitHub

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the resource group you will use in this exercise:

```sh
RESOURCE_GROUP_APP='serverless-RG'
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the Azure region you will use for the deployment:

```sh
LOCATION=$(az group list --query "[?name == 'serverless-RG'].location" --output tsv)
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new variable which value is a randomly generated string that you will use as the name of a new web app:

```sh
WEBAPPNAME2=webapp05022$RANDOM$RANDOM
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new web app using a unique name:

```sh
az webapp create --name $WEBAPPNAME2 --plan serverless-$LOCATION --resource-group $RESOURCE_GROUP_APP --runtime "NODE|9.4"
```

> **Note**: In case the command fails due to duplicate web app name, re-run the last two steps until the command completes successfully  


1. In the **Cloud Shell** pane, click the **Upload/Download files** icon and, in the drop-down menu, click **Upload**. 

1. In the **Open** dialog box, navigate to the **webapp/files** folder, select the **github.json** file, and click **Open**. The file contains the following Azure Resource Manager template:

```json
    {
        "$schema": "http://schemas.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "webAppName": {
                "type": "string"
            },
            "repositoryUrl": {
                "type": "string"
            },
            "branch": {
                "type": "string",
                "defaultValue": "master"
            }
        },
        "resources": [
            {
                "apiVersion": "2015-08-01",
                "type": "Microsoft.Web/sites",
                "name": "[parameters('webAppName')]",
                "location": "[resourceGroup().location]",
                "properties": {},
                "resources": [
                    {
                        "apiVersion": "2015-08-01",
                        "name": "web",
                        "type": "sourcecontrols",
                        "dependsOn": [
                            "[resourceId('Microsoft.Web/Sites', parameters('webAppName'))]"
                        ],
                        "properties": {
                            "RepoUrl": "[parameters('repositoryUrl')]",
                            "branch": "[parameters('branch')]",
                            "IsManualIntegration": true
                        }
                    }
                ]
            }
        ]
    }
```

1. In the **Cloud Shell** pane, click the **Upload/Download files** icon and, in the drop-down menu, click **Upload**. 

1. In the **Open** dialog box, navigate to the **webapp/files** folder, select the **parameters.json** file, and click **Open**. The file contains the following parameters for the Azure Resource Manager template you uploaded previously:

```json
    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "webAppName": {
          "value": "$WEBAPPNAME2"
        },
        "repositoryUrl": {		
          "value": "$REPOSITORY_URL"
        },
        "branch": {
          "value": "master"
        }
      }
    }
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the GitHub repository hosting the web app code:

```sh
REPOSITORY_URL='https://github.com/Azure-Samples/nodejs-docs-hello-world'
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the name of the GitHub repository hosting the web app code and which takes into account any special character the URL might include:

```sh
REPOSITORY_URL_REGEX="$(echo $REPOSITORY_URL | sed -e 's/\\/\\\\/g; s/\//\\\//g; s/&/\\\&/g')"
```

> **Note**: This is necessary because you will use the **sed** utility to insert this string into the Azure Resource Manager template parameters file. Alternatively, you could simply open the file and enter the URL string directly into the file.  

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to replace the placeholder for the value of the **webAppName** parameter with the value of the **$WEBAPPNAME2** variable in the parameters file:

```sh
sed -i.bak1 's/"$WEBAPPNAME2"/"'"$WEBAPPNAME2"'"/' ~/parameters.json
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to replace the placeholder for the value of the **repositoryUrl** parameter with the value of the **$REPOSITORY_URL** variable in the parameters file:

```sh
sed -i.bak2 's/"$REPOSITORY_URL"/"'"$REPOSITORY_URL_REGEX"'"/' ~/parameters.json
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify that the placeholders were successfully replaced in the parameters file:

```sh
    cat ~/parameters.json
```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to deploy the GitHub-resident web app code by using a local Azure Resource Manager template and a local parameters file:

```sh
az group deployment create --resource-group $RESOURCE_GROUP_APP --template-file github.json --parameters @parameters.json
```

1. Wait for the deployment to complete before you proceed to the next task.

> **Note**: The deployment should take about a minute.  

#### View deployment results

1. In the Azure portal, click **Resource groups**.

1. On the **Resource groups** blade, click **serverless-RG**.

1. On the **serverless-RG** blade, click the entry representing the Azure web app you created in the previous task.

1. On the web app blade, click the **Browse** button at the top of the blade.

1. Review the sample Node.js web application deployed from GitHub.

1. Close the new browser tab and return to the browser tab displaying the Azure portal.

## Remove lab resources

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to list all resource groups you created in this lab:

```sh
az group list --query "[?starts_with(name,'serverless')]".name --output tsv
```

1. Verify that the output contains only the resource groups you created in this lab. These groups will be deleted in the next task.

#### Delete resource groups

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the resource groups you created in this lab

```sh
az group list --query "[?starts_with(name,'serverless')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. Close the **Cloud Shell** prompt at the bottom of the portal.


> **Review**: In this exercise, you removed the resources used in this lab.  