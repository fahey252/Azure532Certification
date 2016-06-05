### Design and develop a cloud service
 * With PaaS cloud services, you are entrusting Azure to manage the operating system updates, patching, and deployment lifecycle of your applications.
 * Just like an App Service is hosted on VMs, so too are Cloud Services, however, you have more control over the VMs. You can __install your own software on Cloud Service VMs and you can remote into them__.
 * With Cloud Services, you don't create virtual machines. Instead, you provide a configuration file that tells Azure how many of each you'd like.
 * Unlike VMs created with Azure Virtual Machines, writes made to Cloud Services VMs aren't persistent; there's nothing like a Virtual Machines data disk.
 * Links
   - (Should I choose cloud services or something else?)[https://azure.microsoft.com/en-us/documentation/articles/cloud-services-choose-me/]

#### Install SDKs and install emulators
  * To design and develop cloud services, you need to __install the Azure SDK__. SDK installs many different items, including __emulators, tools, and APIs__ for several different Azure services.
  * The Azure SDK includes the required emulators and Visual Studio templates required to develop cloud services. Any role __template is merely a starting point__ for creating a role implementation. __Could manually write__ the same code for a worker role you implement from scratch.
  * In Visual Studio, Create a new C# Cloud project and choose __Get Microsoft Azure SDK For .NET__ and Download Microsoft Azure SDK.
  * It is important to __stay up to date on the latest SDK__. This allows you to avoid backward compatibility issues with your code base as the SDK evolves.
  * The SDK installs two emulators, the __Azure Compute Emulator and the Azure Storage Emulator__.

#### Develop a web role or worker role
  * When you create a __cloud service project__, you can add one or more cloud service roles (roles) to the project to be included in the deployment package.
  * __Web roles__: Used for web server applications hosted in IIS, such as an ASP.NET MVC application or a Web API application
  * __Worker roles__: Used for running a compute workload. Work in a similar manner to a Windows service.
  * Visual Studio > Visual C#, and then Cloud, and select the __Azure Cloud Service template__ > list of role templates.
  * The cloud service template should be the __startup project in the solution__. References the roles associated with the cloud service and holds the configuration settings for each role.
  * Role Templates (starting point): __ASP.NET Web Role__ (MVC, Web API, Single Page application, Web Forms application, or empty ASP.NET application), __WCF Service Web Role__, __Worker Role__ (empty background worker process where you provide the functionality in Run()), __Worker Role with Service Bus Queue__.
  * Adding __multiple roles will result in multiple VMs__. Each role then operates in isolation with its own configuration settings. It is possible to configure multiple roles to deploy to a single VM.
  * Cloud service web and worker roles are projects that include a class that inherits the __RoleEntryPoint__ interface. Three key methods to override when you implement RoleEntryPoint: __OnStart() (Initialize the environment), Run() (implementation here i.e. process queues) , and OnStop() (Clean up anything you created before the role shuts down.)__. For a worker role, the RoleEntryPoint is the heart of its functionality.
  * __By default, the RoleEntryPoint does nothing__. To create a worker role with any functionality you have to provide custom code in the Run() override
  * Consider putting some key __configuration settings in the role configuration settings__ instead of in the web.config application settings. This makes it possible to __surface those settings to the management portal__ for modification.
  * If your application has any __dependencies__ that require installation on the destination VM or control over IIS-related settings, use a __startup task to provide an unattended deployment__ for this configuration.
  * Any code that relies on the __RoleEnvironment__ global configuration will __fail when the project is not run as a cloud service.__
  * __ServiceDefinition.csdef__: definition for the cloud service, including a list of any __startup tasks__ you add and a definition for each web and worker role.
  * __ServiceConfiguration.(Local|Cloud).cscf__: Configuration settings are edited as you edit the role configuration in the role settings dialog box. Alternate configuration settings, for example for development, test, or production cloud deployments.
  * Can later add the desired roles to the project using either new role templates or __by adding existing projects as roles__.

#### Design and implement resiliency including transient fault handling
  * Design your cloud services to support the potential for increases in server load. __Static Content Hosting pattern, Cache-Aside pattern, Health Endpoint Monitoring pattern, and many more cloud design patterns.
  * Application logic that __accesses remote application services__ requires a form of __retry logic__ to recover from transient connectivity issues.
  * Can leverage the __Transient Fault Handling Application Block__ to assist with this type of implementation for __Azure Storage and Azure SQL Database__.

#### Develop startup tasks
  * Running scripts __prior to starting a role__. Can be a simple __batch file__ that launches a process, adjusts registry settings, runs an MSI (`msiexec.exe`), or invokes Windows __PowerShell scripts__ to configure any number of machine settings, launch a process.
  * Prepare the VM in advance of running your application code.
  * A role enters __“starting” state and does not receive traffic__. Startup tasks run according to their type > __Simple tasks run in order. Background and foreground tasks start asynchronously__ > The RoleEntryPoint type OnStart() is called > role enters __“ready” state and traffic is sent__ to the role endpoints > RoleEntryPoint type Run() method is called.
  * All simple tasks run in sequence __prior to the role being put into ready state__ to start receiving requests.
  * __Foreground tasks__ run asynchronously in parallel to simple and background tasks. The role may be put into ready state after simple tasks are completed but before the foreground or background task has completed. Further, the __role cannot be recycled while a foreground task is still running__.
  * __Background tasks__ run asynchronously in parallel to simple and foreground tasks. The role may be put into ready state after simple tasks are completed but before the foreground or background task has completed. Unlike foreground tasks, __the role CAN be recycled while a background task is still running__.
  * If any of the tasks do __not complete with exit code 0, the role is not started.__
  * Sample batch file:

  ```batch
  @echo off
  regedit.exe /s licensesettings.reg
  exit /b 0
  ```
  * Set the script files in the project to __Copy Always__ so that they will be copied to the __\bin folder__ when the solution is compiled.
  * Add the startup task to the cloud service definition > Open `ServiceDefinition.csdef` > Add a Startup element > commandLine attribute with the name of a batch file to run
  * `executionContext` attribute with the value __“limited” (without administrator privileges) or “elevated.” (change protected resources such as IIS settings, registry)__
  * `taskType` attribute with the value __“simple” (synchronously), “foreground” (run asynchronously in a foreground thread, and the role will not start until the thread completes) or “background” (background thread, in parallel with other background tasks)__.

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <ServiceDefinition name="CloudServiceWorker" xmlns="http://schemas.microsoft.com/
  ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2014-06.2.4">
    <WorkerRole name="Worker" vmsize="Small">
      <Startup>
        <Task commandLine="startup.cmd" executionContext="elevated" taskType="background"/>
      </Startup>
    </WorkerRole>
  </ServiceDefinition>
  ```
  * Startup tasks must be designed to __run more than once for the role__ VM. When roles __restart, the startup tasks run again__. Build tasks to handle restarts gracefully.
  * __Information about the role is not guaranteed__ to be available when the startup task is running. If you must interact with the __RoleEnvironment__ and related information, __use the OnStart()__ override for the RoleEntryPoint.
  * Startup tasks cannot rely on any information related to the role environment, __nor can they interact with the RoleEnvironment object__. To work with __RoleEnvironment, use the OnStart() override in the RoleEntryPoint class__.
  * `RoleEnvironment` class gives configuration, endpoints, and status of running role instances.
  * You could create a startup task that runs as a "simple" task with "elevated" processing to ensure that it runs before the role begins accepting requests (i.e. registry settings).
  * Create a Windows PowerShell script (.ps1) that ensures the .NET Framework 3.5 Windows feature is enabled or other Windows features are set.


