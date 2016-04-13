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

## Create App Service plans

## Migrate Web Apps between App Service plans

## Create a Web App within an App Service plan