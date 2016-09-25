### Monitor and Debug a Cloud Service
  * Logs and performance counter metrics. Can debug locally and remotely.
  * Can remote desktop to the machine to troubleshoot.
  * Look at using monitoring (through the portal) and configuring alertsbased on __endpoint monitoring and host metrics__.

#### Configure diagnostics using the SDK or configuration file
  * __Enabling the Azure Diagnostics__ module in your __service definition__, and then configuring the diagnostic data you want to collect.
  * By default, logs are stored locally on the machine. Can be copied to Azure Storage if configured. Diagnostic data is __not permanently stored__ unless you transfer it to the Microsoft Azure storage emulator or to Azure storage
  * __Diagnostic data sources__: Table storage (Application logs, infrastructure logs, event logs, performance counters) and Blob storage (crash dumps, custom error logs, web server logs, failed request trace logging - large binary data or web related logs (blob))
  * Configuring diagnostics by editing the XML configuration file __diagnostics.wadcfgx__. Using the properties box > Configuration > Enable Diagnostics > Configure. Give Storage Account Credentials to were diagnostics will be copied. 
  * You specify the storage account that you want to use in the _ServiceConfiguration.cscfg_ file using the connection string to the storage account.

  ```xml
  <ConfigurationSettings>
    <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
  </ConfigurationSettings>
  ```

  * You can transfer diagnostic data at scheduled intervals as specified in the configuration. You can incur costs when transferring data to storage account.
  * After deployment, you can re-configure diagnostics within Visual Studio. Select Role > Update Diagnostics. The settings are applied __directly to the running role instead of within your local project__.
  * Can use the existing management portal to configure alerts on select metrics.
  * Can view some diagnostic data using Visual Studio or query storage account directly. VS > Server Explorer > Role > View Diagnostic Data. Can also use third-party tools such as Azure Management Studio.
  * Can no longer configure cloud service diagnostics __programmatically (not supported as of Azure SDK 2.5,).__

#### Profile resource consumption
  * Use Visual Studio (Ultimate or Premium) to determine which fuctions take the most time, features that are CPU or memory intensive and concurrency issues.
  * __CPU Sampling__ (size and events): periodically samples CPU for consumption metrics/call stack. _Lightweight_ and has _little effect_ on the execution of the application methods.
  * __Instrumentation__ (timing): Each _function_ call, entry and exit to see where spending the most time - can isolate to a single module. Investigating _input/output bottlenecks_ such as disk I/O.
    - Elapsed Inclusive:  total time that is spent executing the function or source line
    - Application Inclusive: Time spent in function, excluding time that is spent in calls to the operating system.
    - Elapsed Exclusive: code in the body of the function. Executing functions that are called by the function or source line is excluded.
    - Applicatoin Exclusive: Time in body of function, exclusing inner function calls and operating system calls.
  * __.NET Memory Allocation__: .NET memory allocation - quanity and size of objects. Interrupts the computer processor at _each allocation_ of a .NET Framework object and at _garbage collection_. Can be used in either sampling or instrumentation mode.
  * __Concurrency__: events that block code for running in multi-threaded or multi-process applications. Detailed call stack information every time that _competing threads are forced to wait_ for access to a shared resource. Concurrency visualization data can be _collected only for command line and Windows applications_.
  * __Tier Interaction__: Collects information about __synchronous ADO.NET__ function calls to a SqlServer database.
  * Enable profiling in Visual Studio > deploy cloud service > Settings > Advanced Settings > __Enable Profiling checkbox___ > choose method > Publish.
  * Getting metrics: Servere Explorer > Cloud Services > Right click instance > __View Profile Report__ > downloads and display report in VS.
  * Links
    - [Understanding Performance Collection Methods](https://msdn.microsoft.com/en-us/library/dd264994.aspx)

#### Enable remote debugging
  * Ensure that the deployed cloud service is a __debug build__, includes debug symbols and has the __remote debugger enabled__.
  * Pushlish Wizard > Build Configuration > Debug build. Advanced > __Enable Remote Debugger For All Roles check box.__ > Publish.
  * Server Explorer > Cloud Service > Choose role or instance __Attach Debugger__ > __Attach to Proccess__ > (for example, for a worker role attach to `WaWorkerHost.exe`)
  * Can attached the debugger to all instances in the a role or to an individual instance.
  * When stopped at a breakpoint, you stop all incoming requests. Not a good idea for production. If at breakpoint for a few minutes, traffic is no longer directed to that instance. If you are stopped too long, the remote debugger service (__msvsmon.exe__) detaches from the process.
  * It is not possible to attach the debugger to an entire cloud service; a __role instance must be the target to attach to__.

#### Establish a connection using Remote Desktop cmdlets in Windows PowerShell Azure PowerShell
  * Can enable RDP to get remote into a web or worker instance and debug within. __Can enable RDP when deploying or enable with Powershell/from the Portal after already deployed.__ Download RDP file from the Portal to connect.
  * Portal > Remote > enable for instance or all instances in the role > __Enable Remote Desktop checkbox__ > set username/password > Use existing or create new certificate > Set expiration date for when RDP will be disabled.

  ```powershell
  $cred = Get-Credential  # prompt for username and password
  Set-AzureServiceRemoteDesktopExtension -ServiceName <CloudServiceName> -Credential $cred
  ```
  * Visual Studio > Project Explorer > Publish > __Enable Remote Desktop For All Roles check box__ > Project credentials/expiration date > Ok > Publish.
  * Using RDP > Download RDP file from Portal > Connect > Supply credentials > 

#### Debug using IntelliTrace
  * IntelliTrace is an alternative to remote debugging. IntelliTrace gives you an experience similar to debugging __without the potential to interrupt the service operation__, which can interfere with your ability to reproduce a problem. Requires Visual Studio Ultimate/Enterprise and .NET 4 or 4.5.
  * Work from a __replay of captured events__ that includes environment data and supports __replaying steps__ through the code.
  * Go to specific points in your application's execution (.iTrace file).
  * Use IntelliTrace to review past events that occurred in your solution and the context in which they occurred and navigate to the code (i.e. on Feature Activated).
  * Enable IntelliTrace and publish from Visual Studio. VS > Publish > Advanced > __Enable IntelliTrace checkbox__ > IntelliTrace Settings:
    - Verbosity of __logging__ (General tab): impacts performance
    - .Net __Modules__ (Module tab): opt to collect data from __all modules__, excluding a list of __module name patterns__ or including a list of module name patterns you specify.
    - __Processes__ tab: collect data from all processes, excluding a list of process name patterns or including a list of process name patterns you specify.
  * __Events__: Can choose event categories or individual events.
  * __Size__: control size of data to collect/download from Azure.
  * Can view IntelliTrace logs after deployed using VS. Server Explorer > Cloud Services > role/instance > __View IntelliTrace Logs__ > VS downloads logs and displays summary. __Choose an event or exeception to set a point of reference__.

#### Debug using the Emulator
  * Azure Compute Emulator: run a cloud service locally, attach the debugger, and examine the cloud service configuration, roles, and instances.
  * Default compute emulator: __Emulator Express__. Light-weight, IIS Express, one instance per role, does not require admin permissions to run. 
  * Visual Studio > Properties > Local Development Server: IIS Express, Emulator: Emulator Express > F5 > Show Emulator from system tray.
  * __Storage Emulator__ for tables, queues, blobs. Start from System tray if Compute Emulator is already running or find in Start menu.
  * Configure application to use Storage Emulator (__Must be HTTP__. HTTPS not supported):
    - Blob service: http://127.0.0.1:__10000__/<account-name>/<resource-path>
    - Queue service: http://127.0.0.1:__10001__/<account-name>/<resource-path>
    - Table service: http://127.0.0.1:__10002__/<account-name>/<resource-path>
  * To set connection string: Server Explorer > Role > Properties > Settings > Connection String > __Create Storage Emulation Connection String__ > __Microsoft Azure Storage Emulator option__.

