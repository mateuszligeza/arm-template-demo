# arm-template-demo

This is simple template that can be used to bootstrap your own website in azure.  
It's intended to use with VS Code and it will propt you to install required extensions.  
It creates:
- `Sql server` and `sql database` for the app. It also adds `firewall rule` allowing all [internal azure adresses](https://docs.microsoft.com/en-us/azure/templates/microsoft.sql/2014-04-01/servers/firewallrules#firewallruleproperties-object) to connect to it.
- `Application Insights` for the app (it assumes that app is deployed without application insights nuget package and adds them as website extension to the web app)
- `Web Site` along with the new `App Service Plan`. It adds `Application Insight Extension` to the web site and updates `web config` with the Application Insights instrumentation key

## Architecture diagram

![Architecture diagram](.\img\diagram.png)

## Prerequisites
- Azure Subscription
- [VS Code](https://code.visualstudio.com/download)
  - [VS Code Extension - ARM Template Viewer](https://marketplace.visualstudio.com/items?itemName=bencoleman.armview) - used to visualise the template. Sometimes give parsing error, but it tends to go away on next try ¯\_(ツ)_/¯
  - [VS Code Extension - Azure Resource Manager (ARM) Tools](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools) - this is really esential as it adds IntelliSense to the arm template that is edited  
- [Chocolatey](https://chocolatey.org/docs/installation)

## Deployment Steps
- [Install](https://chocolatey.org/docs/commandsinstall) `Azure CLI` with Chocolatey (requires admin permissions)

```
choco install azure-cli -y
```
> you can use the azure powershell as well `choco install azurepowershell` but it seams that the support for it is [dropping soon (AzureRM is deprecated)](https://docs.microsoft.com/en-au/powershell/azure/new-azureps-module-az?view=azps-3.8.0&viewFallbackFrom=azps-3.7.0) the never version (Az) should be used instead `choco install az.powershell` 

- open PowerShell in `infrastructure` folder

- [Login](https://docs.microsoft.com/en-au/cli/azure/reference-index?view=azure-cli-latest#az-login) to your subscription
```
az login
```
> if you have multiple tenants in your subscription use
> ```
> az login --tenant <tenant-id>
> ```

- [create](https://docs.microsoft.com/en-au/cli/azure/group?view=azure-cli-latest#az-group-create) resource group
```
az group create --name arm-template-demo --location australiaeast
```

- [deploy](https://docs.microsoft.com/en-au/cli/azure/group/deployment?view=azure-cli-latest#az-group-deployment-create) the arm template to create resources
  - option 1: using the parameters file
  ```
  az deployment group create --resource-group arm-template-demo --template-file arm-template.json --parameters arm.parameters.local.json
  ```
  > The `arm.parameters.local.json` file should be added to git ignore, and hold only your local configuration for the development time

  - option 2: using inline parameters
  ```
  az deployment group create --resource-group arm-template-demo --template-file arm-template.json --parameters sqlServerAdminPassword=superSecretPassword!@#456 solutionName=arm-template-demo
  ```
> By default deployment command only adds new resources. If you wish to delete resources that are not defined in the template use `--mode complete` flag

> Database password has to meet the complexity requirements, otherwise the deployment will fail
```
az deployment group create --resource-group arm-template-demo --template-file arm-template.json --parameters arm.parameters.local.json --mode complete
```

## Other useful commands
`az group list` - [list](https://docs.microsoft.com/en-au/cli/azure/group?view=azure-cli-latest#az-group-list) existing resource group

## Additional notes

Resources can be nested. When they are nested we can ommit parent type of the resource type and the parent name in resource name.
```
{
  "type": "Microsoft.Sql/servers",
  "name": "sqlServerName",
  "resources": [
  {
    "type": "databases",
    "name": "sqlDatabaseName",
  ]
}
```
as opposed to 
```
{
  "type": "Microsoft.Sql/servers",
  "name": "sqlServerName",
}
{
  "type": "Microsoft.Sql/servers/databases",
  "name": "sqlServerName/sqlDatabaseName",
},
```
However the ARM viewer extension seems to handle the non-nested syntax better.

---

The website extensions are defined as nuget packages and are referenced by nuget package name. I could not find the list of all available packages as they changed location and nuget was timing out on me. More on that topic can be found below

[Azure Site Extensions (GitHub Wiki)](https://github.com/projectkudu/kudu/wiki/Azure-Site-Extensions)  
[Azure Web Sites Extensions (Azure Blog)](https://azure.microsoft.com/en-au/blog/azure-web-sites-extensions/)  
[Site Extensions are moving to nuget.org by August 2018](https://github.com/Azure/app-service-announcements/issues/87)  
[Application Insights NuGet packages - for Azure](https://docs.microsoft.com/en-us/azure/azure-monitor/app/nuget#additional-packages)  

When using extensions one has to be mindful of the other dependencies as noted in this SO answer: 
[App Insights Status Monitor Extension Failing to deploy with ARM template](https://stackoverflow.com/questions/45106303/app-insights-status-monitor-extension-failing-to-deploy-with-arm-template/45138533)  

---

## Learning resources
[ARM template - docs home page](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/)  
[Structure and syntax of ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-syntax)  
[ARM template functions](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions)  
[Resource definitions](https://docs.microsoft.com/en-us/azure/templates/) - `Most useful reference when writing the template`  
[Azure Quickstart Templates](https://github.com/Azure/azure-quickstart-templates)  
[Azure Resource Manager Schemas](https://github.com/Azure/azure-resource-manager-schemas)  
[Get started with Azure CLI](https://docs.microsoft.com/en-au/cli/azure/get-started-with-azure-cli?view=azure-cli-latest)  
[Azure CLI commands reference](https://docs.microsoft.com/en-au/cli/azure/reference-index?view=azure-cli-latest) - `Most useful reference when figuring out how to script things`  

## Links
[Control Azure services with the CLI](https://docs.microsoft.com/en-us/learn/modules/control-azure-services-with-cli/) - Basic introduction to azre CLI  
[Azure Resource Explorer: a new tool to discover the Azure API](https://azure.microsoft.com/en-au/blog/azure-resource-explorer-a-new-tool-to-discover-the-azure-api/) - introduction to [Resource Explorer](https://resources.azure.com/)  
[Azure ARM Templates for deleting the resources](https://serverfault.com/questions/953965/azure-arm-templates-for-deleting-the-resources)  
[Automating Azure Instrumentation and Monitoring – Part 2: Application Insights](https://blog.kloud.com.au/2018/11/29/automating-azure-instrumentation-and-monitoring-part-2-application-insights/)  
[Deploying a WebApp with Application Insights using ARM](https://winterdom.com/2017/08/01/aiarm)  
[Creating a Go Site Extension and Resource Template for Azure](http://www.wadewegner.com/2015/01/creating-a-go-site-extension-and-resource-template-for-azure/)  
[Differences between Azure CLI products](https://docs.microsoft.com/en-au/cli/azure/cli-versioning-identifiers?view=azure-cli-latest)  
