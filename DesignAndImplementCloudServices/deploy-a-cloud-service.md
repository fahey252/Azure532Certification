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
      - __Incremental__: Minimal impact. Upgrading instances __one upgrade domain at time__. Stop, start, restarts instance. Once all instances in upgrade domain complete, move on to next upgrade domain. Increased availability but takes a __long time__. 5 upgrade domains by default, can be increased to 20. i.e. `<ServiceDefinition upgradeDomainCount="20">`
      - __Simultaneous__: Stop __all__ instances at once. Short amount of time, down time.
      - __Full Deployment__: does not attempt to upgrade the instances. Completely __removes all__ the existing instances and performs a __fresh deployment with the latest package__.
  * Some changes can __not be done via upgrade, needs full deployment__/VIP swap: changing name of role, change upgrade domain count, reduction of local storage sizes.
  * Can deploy an upgrade from Visual Studio or by using the management portal.
  * Visual Studio: Publish Azure Application wizard > Signin/Settings > Deployment Upgrade > select rollout method (incremental) > Publish.
  * Portal: Solution Explorer > Package > Portal > Choose Service > Update > Choose the _.cspkg_ and _.cscfg_ from either your __local machine or a cloud storage account__. 
      - Option: Allow The Update If Role Sizes Change Or The Number Of Roles Change check box: may cause your __role instances to be re-deployed__ to a different host within Azure, and therefore any content written to __local storage may be lost__.
      - Option: Update The Deployment Even If One Or More Roles Contain A __Single Instance__: single instance means role may be __unavailable during upgrade__.

#### VIP swap a deployment
  * Production: VIP and <dns-prefix>.cloudapp.net. Staging: different VIP and <Guid-ID>.cloudapp.net.
  * Conduct a VIP swap operation to re-map the current production VIP and URL to the deployment currently running in the staging environment. Production IP address assigned to the VIP is always maintained.
  * VIPs should swap quickly. Can delete staging slot once deployment is complete.

#### Implement continuous deployment from Visual Studio Online (VSO)
  * Automatically build and deploy a cloud service upon check-in.
  * To connect accounts > Portal > Cloud > Setup Publishing > Authorize Now > OAuth Connection Request > Give Visual Studio Online Domain Name (i.e. fahey.visualstudio.com) > Go to Visual Studio and check in a change > Watch the build status > when build and deployment completes, go to the Portal to see your deployments.
  * Can change slot build deploys to in Visual Studio. Build Definitions > Edit > Process tab > change deployment environment from Proudction to Staging.
  * If you want to delete and redeploy all instances, Process > and set __Allow Upgrade__: false and __Do not Delete__: false.  

#### Implement runtime configuration changes using the portal
  * Custom configuration settings stored in an easily accessible place... not just in the web or worker roles configuration settings. Can be managed in the Portal by others, varying settings per environment.
  * Visual Studio: Double click the cloud project > Settings > give key/value pairs.
  * A setting change in the management portal __fires a changing event__, followed by a changed event. These events are in the __RoleEnvironment__ class. Open the __WorkerRole.cs__ and add event listeners in the __OnStart__ event such: 

  ```c#
  RoleEnvironment.Changing += RoleEnvironmentChanging;

  private void RoleEnvironmentChanging(object sender, RoleEnvironmentChangingEventArgs e) 
  {
    foreach (RoleEnvironmentChange rec in e.Changes)
      {
        // Check for key/values and do something
        RoleEnvironmentConfigurationSettingChange recsc = rec as RoleEnvironmentConfigurationSettingChange;
        if (recsc.ConfigurationSettingName.ToUpper() == “SQLCONNSTRING”)
        {
          // and so on, do stuff
          e.Cancel = true;  // forces role to recycle before changing
        }
      }
  }
  ```
  * Other role events are: __Changed (per instance), Changing, SimultaneousChanging (all role instances at the same time.), SimultaneousChanged, Stopping, StatusCheck (regular interval audit)__. Notice there is no Stopped event...

#### Configure regions and affinity groups