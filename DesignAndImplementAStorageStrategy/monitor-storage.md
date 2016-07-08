### Monitor Storage (metrics and logging)
  * Built-in analytics feature called __Azure Storage Analytics__ used for collecting __metrics__ and __logging__ storage request activity.
  * __Storage Analytics Metrics__ to collect aggregate _transaction_ and _capacity_ data.
  * __Storage Analytics Logging__ to capture _successful_ and _failed_ request attempts to your storage account.
  * Storage Analytics metrics provide the __equivalent of Windows Performance Monitor counters__ for storage services.
  * Storage metrics can be viewed in the management portal. Storage logs can be downloaded and viewed in a reporting tool such as Excel.
  * Links
    - [Monitor, diagnose, and troubleshoot Microsoft Azure Storage](https://azure.microsoft.com/en-us/documentation/articles/storage-monitoring-diagnosing-troubleshooting/)

#### Enable monitoring and logging
  * Storage __metrics and storage logging__ are __not enabled by default__, but you can enable them through the management portal, using Windows PowerShell, or by calling the management API directly.
  * Tables are created for logs. Set the level for each service - blob, queue, files and tables.
  * When you configure Storage Analytics Logging for a storage account, a blob container named __$logs__ is automatically creates a container in blob storage account to store the output of the logs.
    - Convention like: <https://accountname.blob.core.windows.net/$logs/servicetype/YYYY/MM/DD/HHMM/counter.log>.
    - Blob name for each log file does not provide an indication of the time range for the logs.
  * Can also choose which type of requests to log: read, write, or delete. All three are on by default. Use Powershell to set individually. You can log all or individual operations to all storage services (blob, file, table and queue).
  * After Storage Analytics has been enabled, the __log container cannot be deleted;__ however, the contents of the log container can be deleted.
  * Specify the interval for metric collection (__hourly or by minute__).
    - You __can only set minute metrics programmatically__ or by using Windows PowerShell cmdlets.
  * Two levels of service:
      - _Service level_: aggregate statistics for all requests - groups by interval. If no requests, no entry is created.
      - _API level_: Metrics record every request to each service.
  * All requests are included in the metrics collected, __including requests to Storage Analytics__.
  * __Capacity metrics are only recorded for the Blob__ service for the account. Storage size, container size, number of blobs.
  * Table names: __$Metrics(Hour|Minute)PrimaryTransactions(Blob|Table|Queue|File)__ or $MetricsCapacityBlob.
  * In the portal, __set the metrics level for the service to Minimal__ and setup a retention policy.
  * __Minimal__ metrics yield enough information to provide a picture of the __overall usage__ and __health__ of the storage account services.
  * __Verbose__ metrics provide more insight at the __API level__, allowing for deeper analysis of activities and issues, which is helpful for __troubleshooting__.
  * To Enable: Portal > Storage > Metrics > Diagnostics
  * Provide a value for retention according to your retention policy. Will apply to all services. Will apply to logging metrics as well.
  * Can use the __RequestId__ and __operation number__ to uniquely identify an entry to filter duplicate log entries. May have __duplicate log entries__.
  * __Can enable client-side logging__ using Microsoft Azure storage libraries to log activity from client applications to your storage accounts.
  * Logs are stored in a $logs container in Blob storage for your storage account, but the __log capacity is not included in your storage account quota__. A separate 20-terabyte allocation is made for storage logs. 
  * For authenticated calls, all known failed requests are logged, and for __anonymous calls, only failed Get requests for error code 304 are logged__.
  * Links
    - [Enabling Storage Metrics](https://msdn.microsoft.com/en-us/library/azure/dn782843.aspx)

#### Set retention policies and logging levels
  * Retention can be configured for each service in the storage account. Up to __20TB__ of storage, time __0-365 days__. __Will not delete any data__ if retention is not setup.
  * Cannot write new data if storage is filled up.
  * You should use a __shorter retention period for minute metrics__ than hourly metrics because of the significant extra space required for minute metrics.
  * You will also be billed for downloading metrics data from your storage account.
  * If you disable metrics, __all metrics previously collected will be retained__ until the retention period expires

  ```powershell
  # unlimited retention
  Set-AzureStorageServiceMetricsProperty –MetricsType Minute –ServiceType Blob –MetricsLevel Service –RetentionDays 0   
  
  # 90 days retention
  Set-AzureStorageServiceMetricsProperty –MetricsType Hour –ServiceType Blob –MetricsLevel ServiceAndApi –RetentionDays 90
  Set-AzureStorageServiceLoggingProperty –ServiceType Blob –LoggingOperations read,write,delete –RetentionDays 90

  # to disable
  Set-AzureStorageServiceMetricsProperty –ServiceType Blob –MetricsLevel None
  Set-AzureStorageServiceLoggingProperty –ServiceType Blob –LoggingOperations none
  ```

#### Analyze logs
  * Storage Analytics metrics are collected in tables.
  * Can view metrics in both portals.
  * Portal > __Monitor__ > Set metric > May __choose up to six metrics__ to show in the monitoring graph.
  * Can choose relative or absolute metrics as well as time frames (__6 hours, 24 hours, 7 days__).
  * Predefined metrics, including __total requests, total egress, average latency, and availability __.
  * Logs are stored as block blobs in delimited text format.
  * You’ll find entries for authenticated and anonymous requests. Will cause differnt log information to be logged. Logs track things such as if request was billable, used a SAS token, throttling, requests for analytics, etc.
  * Can view logs with __Powershell, AzCopy (included in Azure SDK), Excel__.
  * Can find information like request from IP address range, SAS token usage, containers/frequency of access, network errors.
  * Can run the __Azure HDInsight Log Analysis Toolkit__ (LAT) for a deeper analysis of your storage logs.
  * Configuring alerts based on latency and availability when using monitoring.

  ```powershell
  Get-AzureStorageBlob -Container '$logs' |
    where {
      $_.Name -match 'blob/2014/12/01/09' -and
      $_.ICloudBlob.Metadata.LogType -match 'write'
    } |
    foreach {
      "{0} {1} {2} {3}" –f $_.Name,
      $_.ICloudBlob.Metadata.StartTime,
      $_.ICloudBlob.Metadata.EndTime,
      $_.ICloudBlob.Metadata.LogType
    }
  ```

