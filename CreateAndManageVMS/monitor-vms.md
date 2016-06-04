### Monitor VMs
  * Analyzing metrics as well as collecting log data from system log files and from applications running within the VM. Status of your VMs, their __resource utilization, their operational health, and diagnostic details__.

#### Configure endpoint monitoring
  * Azure itself makes __GET requests__ against these endpoints periodically (and from configurable locations around the world) and exposes additional metrics that report on the __latency and availability experienced__.
  * Portal: Monitoring blade > Provide a short name for the endpoint and the __URL you want Azure to ping__. Also pick __one to three locations__ from which to ping.
  * To add an endpoint with PowerShell, use the `Add-AzureEndpoint` CmdLet.

#### Configure alerts
  * After your VM is configured to collect metrics or monitor endpoints, you can configure alerts that __send an email when a particular threshold is crossed__.
  * Portal: (Management Services|Operations) > Alert blade > Add Rule > Define Alert > select Virtual Machine, and for the service name, select the cloud service containing the VM for which you want to configure an alert > select the metric to define your threshold condition.
  * __Add Rule, add condition, add threshold, choose period of time to compute the rule, choose notification method (email)__.
  * You can also setup Alerts to use metrics computed over an aggregate time period (i.e. 5 minutes).
  * For an alert rule on events, a __rule can send a notification on every event, or, only when a certain number of events happen__.
  * Can configure __webhooks__ on your Alerts to route notifications to various channels

#### Configure diagnostic and monitoring storage location
  * VMs come installed with __Azure Virtual Machine Agent__, which installs and manages extensions running within your VM.
  * __Out of box metrics include: Disk read, disk write, CPU percentage, Network in, network out.__ Do not get the guest operating system values by default (A guest system (guest operating system) is a virtual guest or virtual machine (VM) that is installed under the host operating system. The guests are the VMs that you run in your virtualization platform.)
  * When enable diagnostics, the __IaaSDiagnostics__ extension is installed and used to collect additional metrics. Includes .NET metrics, Windows event logs: __system, security, and application__ logs, IIS Logs etc.
  * Diagnostics are collected to tables within an Azure Storage account that you designate. Metric data is written to the `WADPerformanceCountersTable` and aggregated by the minute (WADMetricsPT1M table) and hourly (WADMetricsPT1H table). WADWindowsEventLogsTable. You can also set up custom
  * The __IIS logs__ are different in that they are __written as blobs to Blob__ storage under the `wad-iis-log-files` container.
  * Portal: Create New Vm > Optional Configuration > Diagnostics > Turn on > Select an Azure Storage account. If VM already exists, go to the Montitoring blade to turn on diagnostics.
  * Currently, monitoring and diagnostics data beyond disk read, disk write, CPU percentage, and network in and network out __cannot be enabled or viewed for Linux VMs__.
  * __IIS logs can be retrieved only from Blob storage__, not Table storage. __Diagnostics logs, Windows System, Performance Counters use table storage.__

#### Monitoring metrics
  * You can assess the status and health of your VM by viewing its metrics in the __portal, by querying table storage for diagnostic logs, or by downloading IIS logs from Blob storage__.
  * Can setup charts in the portal by choosing the Monitoring blade.
  * You can view Windows event logs, the diagnostic infrastructure logs, and application logs by querying their respective tables (__WADWindowsEventLogsTable, WADDiagnosticInfrastructure-LogsTable, WADLogsTable__) in Table storage.  Visual Studio > Server Explorer > Azure account > Storage > Expand Tables > right click and query.
  * IIS logs can be retrieved from Blob storage using the tool of your choice. Visual Studio > Server Explorer > Azure > Storage > Blobs > Right-click `wad-iis-logs` and select View Blob Container to display its contents.


 
