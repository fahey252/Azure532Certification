### Deploy a Cloud Service

#### Package a deployment
  * Includes all of the application and service model files needed to deploy and run an application. Deploy two files: the __application package file (\*.cspkg)__ and the __service configuration file (*.cscfg)__.
  * You can also use the __CSPack command-line tool__ to create a package from the command line. Installed with the Azure SDK. CSPack uses the contents of the service definition file and service configuration file to define the contents of the package.

  ```cmd
  > cd C:\Program Files\Microsoft SDKs\Azure\.NET SDK\[sdk-version]\bin\
  > cspack [DirectoryName]\[ServiceDefinition]
       /role:[RoleName];[RoleBinariesDirectory]
       /sites:[RoleName];[VirtualPath];[PhysicalPath]
       /out:[OutputFileName]
  ```

#### Upgrade an automatic, manual, or simultaneous deployment
  * How Azure should roll out changes to the role instances when re-deploying. 
      - __Incremental__: Minimal impact. Upgrading instances __one upgrade domain at time__. Stop, start, restarts instance. Once all instances in upgrade domain complete, move on to next upgrade domain. Increased availability but takes a __long time__. 5 upgrade domains by default, can be increased to 20. i.e. `<ServiceDefinition upgradeDomainCount=20>`
      - __Simultaneous__: Stop __all__ instances at once. Short amount of time, down time.
      - __Full Deployment__: does not attempt to upgrade the instances. Completely __removes all__ the existing instances and performs a __fresh deployment with the latest package__.
  * Some changes can __not be done via upgrade, needs full deployment__/VIP swap: changing name of role, change upgrade domain count, reduction of local storage sizes.
  * Can deploy an upgrade from Visual Studio or by using the management portal.
  * Visual Studio: Publish Azure Application wizard > Signin/Settings > Deployment Upgrade > select rollout method (incremental) > Publish.
  * Portal: Solution Explorer > Package > Portal > Choose Service > Update > Choose the _.cspkg_ and _.cscfg_ from either your __local machine or a cloud storage account__. 
      - Option: Allow The Update If Role Sizes Change Or The Number Of Roles Change check box: may cause your __role instances to be re-deployed__ to a different host within Azure, and therefore any content written to __local storage may be lost__.
      - Option: Update The Deployment Even If One Or More Roles Contain A __Single Instance__: single instance means role may be __unavailable during upgrade__.

#### VIP swap a deployment

#### Implement continuous deployment from Visual Studio Online (VSO)

#### Implement runtime configuration changes using the portal

#### Configure regions and affinity groups