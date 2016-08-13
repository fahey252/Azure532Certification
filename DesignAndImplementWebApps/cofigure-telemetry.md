#Configure Diagnostics, Monitoring, and Analytics
 * Be sure to enable logging and / or tracing for the App Service. App Settings > Diagnostics logs
 * Links
    - [Monitor Web Apps in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/web-sites-monitor/)
    - [Enable diagnostics logging for web apps in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/web-sites-enable-diagnostic-log/)
    - [Troubleshoot a web app in Azure App Service using Visual Studio](https://azure.microsoft.com/en-us/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio/)

##Retrieve Diagnostics Data

  * Log files can be downloaded using either FTP, Azure PowerShell, or the Azure CLI. Create deployment credentials to get access.
  * FTP - download an FTP client, connect and copy logs to local computer
  * Powershell
  ```powershell
  Save-AzureWebSiteLog -Name webappnam  # log.zip saved to current directory
  Get-AzureWebSiteLog -Name webappname -Tail  # streaming
  ```

  * CLI
  ```
  $ azure site log download webappname		# log.zip saved to current directory
  $ azure site log tail webappname	# stram of logs
  ```

  * Log type file locations
  	- Application Logging: /LogFiles/Application/		Text files.
  	- Failed Request Tracing:  /LogFiles/W3SVC#########/  Has XSL and XML file. View in Internet Explorer.
  	- Detailed Error Logging: /LogFiles/DetailedErrors/  Many .html for any HTTP errors. Gives detailed messages, likely causes, things you can try, helpful resources/links.
  	- Web Server Logging: /LogFiles/http/RawLogs  W3C extended log format. Use a log parser to view such as Log Parser 2.2
  * Can view logs with __Visual Studio Application Insights__. Add the Application Insights SDK to your project in Visual Studio. Right click your project and choose Manage NuGet Packages. Select `Microsoft.ApplicationInsights.TraceListener`
  * Logs generally take this when saving to the file system.
  ```
  {Date}  PID[{process id}] {event type/level} {message}
  ```
  * When logging to table storage, additional properties are used to facilitate searching the data stored in the table as well as more granular information on the event.
  * When logging to blob storage, data is stored in comma-separated values (CSV) format.

##View streaming logs, configure endpoint monitoring, configure alerts, configure diagnostics, use remote debugging, monitor Web App resources
  * Streaming Logs
  ```
  Get-AzureWebSiteLog -Name webappname -Tail # streaming
  Get-AzureWebSiteLog -Name webappname -Tail -Message Error		#for specific events
  Get-AzureWebSiteLog -Name webappname -Tail -Path http		#additional filters
  // or
  $ azure site log tail webappname
  $ azure site log tail webappname --filter Error
  $ azure site log tail webappname --path http
  ```
  
  * Notes
  	- Log streaming will also stream information written to any text file stored in the D:\home\LogFiles\ 
  	- While developing an application, it is often useful to see logging information in near-real time. 
  * Configure Endpoint Monitoring
  	- Monitoring functionality for Standard and Premium App Service plans via the Monitor management page. Monitor up to 2 endpoints from up to 3 geographic locations.
  	- Configures web tests from geo-distributed locations that test response time and uptime of web URLs. Runs every 5 minutes.  
  	-  Uptime is considered 100% when the response time is less than 30 seconds and the HTTP status code is lower than 400. Configure an alert if failure occurs.
  	-  Setup, go to Settings > Monitoring > Add a name > Add endpoint to monitor such as site.com/status > Choose several locations requests come from.
  	-  Setup alert: Management Services > Add Rule > Metric > Response time > add metric value.
  * Configure Alerts
  	- Can setup alerts for when metric rules are exceeded.
  	- For an alert rule on events, a rule can send a notification on every event, or, only when a certain number of events happen.
  	- Choose a resource > Alert Rules > Give name/description > Choose a Metric or Event > Choose threshold/length of period.
  	- Can manage metrics created by viewing threshold compared to the previous day to get an idea of how many emails would be sent. 
  	- __Webhooks__ allow the user to route the Azure Alert notifications to other systems for post-processing or custom notifications so not just emails. Given a URL endpoint to POST to, authentication with token and then configure the payload to send when the metric/event happens. 
  * Configure Diagnostics
  	- Diagnostics are enabled on the Configure tab for the web app. There are __application__ (by the application) and __site__ (by the web server) diagnostic types.
  	- __Application Logging__ (File system, table, blob storage)
  		+ Logging of information produced by the application. Can set for __File System__ (stored on the file system, get via FTP), __Table Storage__ (must have a storage account, accessed via Storage Client), or __Blob Storage__ (need storage account, storage container and blob name.)
  		+ Logging Level field determines whether __Error, Warning, or Information__ level information is logged. You may also select Verbose, which will log all information produced by the application.
  		+ Note: Application logging to table or blob storage is only supported for .NET applications.
  		+ All logging locations can be on at the same time with different levels. Log errors and warnings to storage as a long-term logging solution, while enabling file system logging with a level of verbose after instrumenting the application code in order to troubleshoot a problem.
  		+ Diagnostics can be enabled from Azure PowerShell using the __Set-AzureWebsite__
  	- __Site Logging__
  		+ Logging of web requests, failure to serve pages, or how long it took to serve a page. Can save to file system or storage account.  Specify a quota for file system logging. The minimum size is 25MB and the maximum is 100MB. The default size is 35MB
  		+ Web server logs are never deleted from blob storage. Specify a retention period in number of days.
  		+ __Detailed Error Messages__ - Turn on detailed error logging to log additional information about HTTP errors (status codes greater than 400). Detailed Error Messages and __Failed Request Tracing__ place significant demands on a web app. Turning off when done troubleshooting.
  		+ Can setup advanced configs for location, buffer sise and folder size via App Settings __(DIAGNOSTICS_TEXTTRACELOGDIRECTORY, DIAGNOSTICS_TEXTTRACEMAXBUFFERSIZEBYTES, DIAGNOSTICS_TEXTTRACEMAXLOGFOLDERSIZEBYTES)__.
  	- Can write an application log with:
  	```
	System.Diagnostics.Trace.TraceError("Something bad happened");

  	```

  * Use Remote Debugging
  	- Visual Studio tools help debug a web app while it runs in App Service, by running in debug mode remotely or by viewing application logs and web server logs. Includes debugging WebJobs.
  	- Can connect to your AppService directly in Visual Studio and update the `web.config` and other files for real time updates.
  	- You can set breakpoints, manipulate memory directly, step through code, and even change the code path with paid for versions of Visual Studio.
  	- Do a Debug configuration deploy then choose Attach Debugger or Attach to Process.  Break points will then be hit in VS. 
  	- For WebJobs, deploy the project with a Debug build.  Find the WebJob in the Server Explorer, right click and choose Attach Debugger.  Can see status of the WebJob by right clicking and choosing View Dashboard.
  	- Running in debug mode in production is not recommended. If your production web app is not scaled out to multiple server instances, debugging will prevent the web server from responding to other requests. With multiple servers, requests aren't gauranteed to go to the server you are attached to. For troubleshooting production problems, your best resource is application tracing and web server logs.
  	- Make sure that the debug attribute of the compilation element in the Web.config file is set to true.
  	```xml
  	  <compilation debug="true" targetFramework="4.5" />
	```

  * Monitor Web App Resources
  	- Metric retention policies: Minute granularity metrics are retained for 24 hours, Hour granularity metrics are retained for 7 days, Day granularity metrics are retained for 30 days.  __Minute -> 1 Day, Hour -> 1 Week, Day -> 1 Month___
    - To add metrics click on Monitoring in Settings then add metrics.
  	- For scaling, apps can run in __Standard__ or __Shared__ mode and can monitor ussage quotas. For __Usage Quotas__ each Azure subscription has access to a pool of resources provided for the purpose of running up to 100 web apps per region in Shared mode.  When a web app is configured to run in Standard mode, it is allocated dedicated resources equivalent to the Small (default), Medium or Large.  There are no limits to the resources a subscription can use for running web apps in Standard mode.
  	- Usage Quotas are based on
  		+ __Data Out, CPU Time, and Memory__: Will shutdown app when exceed and start up when interval is up.
  		+ __File System Storage__: Will move to read only when exceed. Will resume when more storage is avialable.
  		+ __Linked Resources__:  Databases.  Various details
  	- Quotas are not a matter of performance or cost, but it's the way Azure governs resource usage in a multitenant environment by preventing tenants from overusing shared resources. Exceeding means downtime or reduce functionality. Perhaps move to a higher App Service plan.
 