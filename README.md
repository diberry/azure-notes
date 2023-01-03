# azure-notes

## Web properties

* [JavaScript at Microsoft](https://developer.microsoft.com/en-us/javascript/)
* [Learn (content)](https://learn.microsoft.com/)
* [Tech Communities](https://techcommunity.microsoft.com/)
* [Learn skills for jobs](https://opportunity.linkedin.com/skills-for-in-demand-jobs)
   * [Career Essentials in Software Development](https://www.linkedin.com/learning/paths/career-essentials-in-software-development)

## Azure

* [Azure resource explorer](https://resources.azure.com/)
 
* Easy auth:  
   * In resource explorer, location for easy auth is `/providers/Microsoft.Web/sites/APP-SERVICE-NAME/config/authsettingsV2/list?api-version=2020-12-01`
   * Login: https://APP-SERVICE-NAME.azurewebsites.net/.auth/login/aad
   * Logout: https://APP-SERVICE-NAME.azurewebsites.net/.auth/logout
   * loginParameters for easy auth single app only: ["response_type=code id_token","scope=openid offline_access profile"]
   * loginParameters for easy auth single app only + Graph: ["response_type=code id_token","scope=openid offline_access profile https://graph.microsoft.com/User.Read"]
   * loginParameters for easy auth single app only + Graph + 2nd app's API: ["response_type=code id_token","scope=openid offline_access profile https://graph.microsoft.com/User.Read api://SECOND-APP-CLIENT-ID/user_impersonation"]

  ```
  "identityProviders": {
        "azureActiveDirectory": {
          "enabled": true,
          "registration": {
            "openIdIssuer": "https://sts.windows.net/51397421-87d6-42c1-8bab-98305329d75c/v2.0",
            "clientId": "4480f5c3-01a7-426a-b602-dba4e7b3f776",
            "clientSecretSettingName": "MICROSOFT_PROVIDER_AUTHENTICATION_SECRET"
          },
          "login": {
            "loginParameters": [
              "response_type=code id_token",
              "scope=openid offline_access profile https://graph.microsoft.com/User.Read"
            ],
            "disableWWWAuthenticate": false
          },
          "validation": {
            "jwtClaimChecks": {},
            "allowedAudiences": [],
            "defaultAuthorizationPolicy": {
              "allowedPrincipals": {}
            }
          }
        }}        
        ```
        
## Azure CLI

### Current logged-in user

```
# acquire logged in user context
export CURRENT_USER=$(az account show --query user.name -o tsv)
export CURRENT_USER_OBJECTID=$(az ad user show --id $CURRENT_USER --query objectId -o tst)
```

## Active Directory

### App registrations

* Error:  "The directory object quota limit for the Principal has been exceeded. Please ask your administrator to increase the quota limit or delete objects to reduce the used quota." - [StackOverflow showing PowerShell command to fix](https://stackoverflow.com/questions/58935129/cant-create-new-service-principals-in-azure-despite-being-under-quota)

## Azure App Service

* Easy auth settings
  * WEBSITE_AUTH_CLIENT - don't create - is created and hidden when you configure easy auth
  * MICROSOFT_PROVIDER_AUTHENTICATION_SECRET - set this to the app registration secret
  * WEBSITE_AUTH_TENANT_ID - this is either your tenant or "COMMON", might be another name depending on how the config is programmed in JS
* Port
  * 8080 is default
  * change via App Setting ->  WEBSITES_PORT
* Install NPM packages after Zip deploy
  * App Setting -> SCM_DO_BUILD_DURING_DEPLOYMENT -> true
* Configure logging to container logs
  * Monitoring -> Logs -> Enable
  * Download lixux logs: https://YOUR-RESOURCE-NAME.scm.azurewebsites.net/api/logs/docker/zip
* Configure application insights
  * Settings -> Configuration -> Turn on Application Insights   
* Configure easy auth:
  * Add Authentication with Microsoft Identity provider, copy client id to notepad (id and secret are already in app settings for you)
  * Configure `authsettingsV2` in Azure Resource Explorer (link above) to add the `login` section
      ```
      "identityProviders": {
          "azureActiveDirectory": {
            "enabled": true,
            "login": {
              "loginParameters":[
                "response_type=code id_token",
                "scope=openid offline_access profile https://graph.microsoft.com/User.Read"
              ]
            }
          }
        }
      },
      ```
* View logs
    * container startup logs: /Logs/*_docker.log
    * runtime logs (console.log: /Logs/*_default_docker.log
    * easyauth: /Logs/*_easyauth_docker.log

## Azure Cloud shell

[Azure cloud shell](https://ms.portal.azure.com/#cloudshell/) allows you to use Azure CLI without having to install it. 

* Has [jq](https://stedolan.github.io/jq/) (commandline JSON processor) installed

## Cognitive Services

### Content moderator

* [Sample image](https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg)

## Databases

* [AdventureWorks](https://docs.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)
* [Convert SQL API JSON to MongoDB API BSON](https://gist.github.com/seesharprun/c36fcc13c1c5766a4ae38439729dbe65) - .NET gist
   * [Cosmisworks data](https://github.com/seesharprun/cosmicworkstool/tree/main/src/node/data) - customers and products 
   * [Customers.json](https://gist.github.com/seesharprun/c36fcc13c1c5766a4ae38439729dbe65#file-customers-json)
   * [Products.json](https://gist.github.com/seesharprun/c36fcc13c1c5766a4ae38439729dbe65#file-products-json)
* Azure SDK for Cosmos DB
   * [Families.json](https://github.com/Azure/azure-sdk-for-js/blob/main/sdk/cosmosdb/cosmos/samples/v3/javascript/Data/Families.json)
* [CosmicWorks](https://github.com/azurecosmosdb/cosmicworks) - GitHub repo
* [cosmos-db-mongodb-api-javascript-samples](https://github.com/Azure-Samples/cosmos-db-mongodb-api-javascript-samples) - GitHub repo 
* [MongoDB aggregations book](https://www.practical-mongodb-aggregations.com/)
* [Cognitive Search 10K books](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/main/search-website/bulk-insert/good-books-index.json)

### Cosmos DB

* Server-side JS - [Azure/azure-cosmosdb-js-server](https://github.com/Azure/azure-cosmosdb-js-server)
* [Node sp sample](https://github.com/Azure/azure-cosmosdb-node/tree/master/samples/ServerSideScripts)

## Database emulators

* [Cosmos DB from container for linux](https://docs.microsoft.com/en-us/azure/cosmos-db/linux-emulator?tabs=sql-api%2Cssl-netstd21#run-the-linux-emulator-on-macos)

## Debug SAS tokens

* Create SAS token in portal then compare to SAS token created with generateBlobSASQueryParameters

## Functions

### host.json

* Timer trigger won't run if you are logging to Application Insights with too high a sampling rate
* Timer trigger may stop working if functions run past default timeout (set the default timeout explicitly)
* Review "Diagnose and solve problems" in portal to find issues

### Custom `route` in `function.json`

```json
{
    "bindings": [
    {
        "type": "httpTrigger",
        "name": "req",
        "direction": "in",
        "methods": [ "get" ],
        "route": "products/{category:alpha}/{id:int?}"
    },
    {
        "type": "http",
        "name": "res",
        "direction": "out"
    }
    ]
}
```

### Blob trigger settings in host.json

Notice that blob trigger follows the queue trigger settings.

```
{
    "version": "2.0",
    "extensions": {
        "queues": {
            "maxPollingInterval": "00:00:02",
            "visibilityTimeout" : "00:00:30",
            "batchSize": 16,
            "maxDequeueCount": 5,               // retries the function for that blob 5 times by default
            "newBatchThreshold": 8,
            "messageEncoding": "base64"
        }
    }
}

```

### Azure Functions - GitHub action

* [Azure Functions Action](https://github.com/marketplace/actions/azure-functions-action)

```
- name: 'Run Azure Functions Action'
  uses: Azure/functions-action@v1
  id: fa
  with:
    app-name: 'AdvocacyGithubTraffic'
    slot-name: 'Production'
    package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/output'
    publish-profile: ${{ secrets.AZUREAPPSERVICE_PUBLISHPROFILE_6F0DB2747FBF4EFFADD6A4472638F303 }}
```

## GitHub

### Actions

#### Ignore node_modules in artifact

```yaml
- name: Upload artifact for deployment job
  uses: actions/upload-artifact@v2
  with:
    name: ${{ secrets.DEPLOY_APP_NAME }}
    path: |
         .
         !./node_modules
```

* Found in [basic-express-typescript](https://github.com/dfberry/basic-express-typescript/blob/main/.github/workflows/deploy-to-stage.yml)
* Combine with App Service deployment
    * Install NPM packages after Zip deploy
    * App Setting -> SCM_DO_BUILD_DURING_DEPLOYMENT -> true 
## Learn sandbox

If learn sandbox doesn't let you in, recreate a new one which resets. 
* Must switch tenant to Sandbox tenant in both portal and VSCode

The Learn sandbox subscription has the following name and tenant ID:

* Name: Concierge Subscription
* Tenant ID: 604c1504-c6a3-4080-81aa-b33091104187

## Microsoft Graph

### My profile from SDK

```javascript
// package.json - type: "module"

import graph from '@microsoft/microsoft-graph-client';
import 'isomorphic-fetch';

const getAuthenticatedClient = (accessToken) => {
    // Initialize Graph client
    const client = graph.Client.init({
        // Use the provided access token to authenticate requests
        authProvider: (done) => {
            done(null, accessToken);
        }
    });

    return client;
}

// https://developer.microsoft.com/en-us/graph/graph-explorer
// https://jwt.ms/
// https://github.com/Azure-Samples/ms-identity-easyauth-nodejs-storage-graphapi/blob/main/2-WebApp-graphapi-on-behalf/controllers/graphController.js

const main = async (accessToken) => {


    try {
        const graphClient = getAuthenticatedClient(accessToken);

        const profile = await graphClient
        .api('/me')
        .get();

        return profile;

    } catch (err) {
        throw err;
    }
}

const accessToken = "... replace with your access token ...";

main(accessToken).then((userData)=>{
    console.log(userData);
}).catch((err)=>{
    console.log(err);
})
```

### My profile from REST

```javascript
// package.json - type: "module"

import axios from 'axios';


// https://developer.microsoft.com/en-us/graph/graph-explorer
// https://jwt.ms/

const main = async (accessToken) => {


    try {

        const url = 'https://graph.microsoft.com/v1.0/me';

        const options = {
            method: 'GET',
            headers: {
                Authorization: 'Bearer ' + accessToken,
                'Content-type': 'application/json',
            },
        };

        const graphResponse = await axios.get(url, options);

        const { data } = await graphResponse;
        return data;

    } catch (err) {
        throw err;
    }
}

const accessToken = "... replace with your access token ...";

main(accessToken).then((userData)=>{
    console.log(userData);
}).catch((err)=>{
    console.log(err);
})

```

## Visual Studio Code

## Docker containers for dev containers

* [GitHub templates of containers](https://github.com/microsoft/vscode-dev-containers)

### Log issue against an Azure extension

1. Look up extension in [marketplace](https://marketplace.visualstudio.com/items)
2. On extensions page on marketplace, find source code repo under project details
3. Open issue on repo

### Debug with current file

In `./.vscode/launch.json` file:

```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "skipFiles": [
                "<node_internals>/**"
            ],
            // Use the ${file} variables
            "program": "${workspaceFolder}\\${file}"
        }
    ]
}
```

### Debug with external terminal

In `./.vscode/launch.json` file:

```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "program": "${workspaceFolder}\\${file}",
            // Use this line to indicate an external terminal - such as reading into program from user input
            "console": "externalTerminal"
        }
    ]
}
```

### Debugging Azure Functions

#### Error: Can't find task for func: host start

* Always run functions app in docker container because Functions runtime are directly tied to Node runtime
* Make sure Azure Functions extensions in installed and _loaded_ in VS Code in the container. The extenion may be in ./vscode/extensions.json, but may not be loaded correctly
