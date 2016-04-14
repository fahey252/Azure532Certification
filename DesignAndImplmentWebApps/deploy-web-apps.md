# Deploy Web Apps

## Define deployment slots
  * Deploy staging deployment slot before swapping it with the production slot. Different version of app to different URLs and then swap between slots. (-dev, -staging, etc.)
  * Eliminates downtime(warm up), traffic redirection, can be __Auto Swapped__
  	- Auto Swap is good for continuous deploy once servers are warmed up.
  * When slot configurations are cloned, some settings get cloned (slot specefic such as app configurations, connection strings) others do not (certificates).  Can mark configurations as slot specific.
  * Make sure slot settings are correct before swapping with production!
  * Multi-phase swap: swap configuration to see what happens first, then code.
  ```powershell
  New-AzureRmWebAppSlot -ResourceGroupName [resource group name] -Name [web app name] -Slot [deployment slot name] -AppServicePlan [app service plan name]

  ```

  * Links
  	- [Set up staging environments for web apps in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/web-sites-staged-publishing/)
  
## Roll back deployments
  * Any errors are identified in production after a slot swap, roll the slots back to their pre-swap states by swapping
  * Some apps may require custom warm-up actions. The `applicationInitialization` configuration element in `web.config` allows you to specify custom initialization actions to be performed before 
  * Delete a deployment slot

  ```powershell
  Remove-AzureRmResource -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots –Name [web app name]/[slot name] -ApiVersion 2015-07-01
  ```

  * Swap a deployment
  ```bash
  $ azure site swap webappslotstest
  ```

## Implement pre and post deployment actions
  * [Kudo](https://github.com/projectkudu/kudu/wiki) is the engine behind git deployments in Azure Web Apps
  * Add an App Setting of `POST_DEPLOYMENT_ACTION` or `PRE_DEPLOYMENT_ACTION` and set the value to the patch of the .cmd, .ps1 or batch file to run after deployment
  * Can have multiple scrips run by setting the `SCM_POST_DEPLOYMENT_ACTIONS_PATH` setting.  The default path is `site\deployments\tools\PostDeploymentActions`
  * Can override the default deployment actions with having a `.deployment` files in root directory
  * Can use custom deployment script generator/`.deployment` file by issueing `azure site deploymentscript [options]`

## Create, configure and deploy a package
  * **Quick Create** and **Custom Create** are two waysto create and deploy a cloud service package
  *  Quick Create, you can create a cloud service just by specifying its URL and the region where it will be physically hosted. If you choose Custom Create, you can immediately publish a cloud service by specifying a package (.cspkg file), a configuration (.cscfg) file, and a certificate.
  * A cloud service is created from three components, the service definition (.csdef), the service config (.cscfg), and a service package (.cspkg). Both the ServiceDefinition.csdef and ServiceConfig.cscfg files are XML-based and describe the structure of the cloud service and how it's configured; collectively called the model. The ServicePackage.cspkg is a zip file that is generated from the ServiceDefinition.csdef and among other things, contains all of the required binary-based dependencies. Azure creates a cloud service from both the ServicePackage.cspkg and the ServiceConfig.cscfg.
  * __Service Definition__ The cloud service definition file (.csdef) defines the service model, including the number of roles.
  * __Service Configuration__ The cloud service configuration file (.cscfg) provides configuration settings for the cloud service and individual roles, including the number of role instances. The configuration values for the cloud service can be changed while the cloud service is running.
  * __Service Package__ The service package (.cspkg) contains the application code and configurations and the service definition file
  * The service definition file defines the runtime settings for your application including what roles are required, endpoints, and virtual machine size. The service configuration file configures how many instances of a role are run and the values of the settings defined for a role.
  * May need to configure certificates, if role instances (VMs) allow remote deskop as well as monitoring with Azure Diagnostics.
  * To upload a package, create new Cloud Service -> then upload the `.cspkg` file and `.cscfg` file. If the package was signed, upload the certificates `.pfx` file and provide its password.
  * You can use the __CSPack__ command-line tool (installed with the Azure SDK) to create the package file as an alternative to Visual Studio
  * Links
  	- <https://azure.microsoft.com/en-us/documentation/articles/cloud-services-how-to-create-deploy-portal>
  	- <https://azure.microsoft.com/en-us/documentation/articles/cloud-services-model-and-package>
  	- <https://azure.microsoft.com/en-us/documentation/articles/vs-azure-tools-publishing-a-cloud-service>
  	- <https://azure.microsoft.com/en-us/documentation/articles/vs-azure-tools-azure-project-create>
  	- <https://azure.microsoft.com/en-us/documentation/articles/vs-azure-tools-cloud-service-publish-set-up-required-services-in-visual-studio>

## Create App Service plans

## Migrate Web Apps between App Service plans

## Create a Web App within an App Service plan