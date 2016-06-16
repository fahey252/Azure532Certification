### Monitor and Debug a Cloud Service
  * Logs and performance counter metrics. Can debug locally and remotely.
  * Can remote desktop to the machine to troubleshoot.

#### Configure diagnostics using the SDK or configuration file
  * __Enabling the Azure Diagnostics__ module in your __service definition__, and then configuring the diagnostic data you want to collect.
  * By default, logs are stored locally on the machine. Can be copied to Azure Storage if configured. Diagnostic data is __not permanently stored__ unless you transfer it to the Microsoft Azure storage emulator or to Azure storage
  * __Diagnostic data sources__: Table storage (Application logs, infrastructure logs, event logs, performance counters) and Blob storage (crash dumps, custom error logs, web server logs, failed request trace logging - large binary data or web related logs (blob))
  * Configuring diagnostics by editing the XML configuration file __ diagnostics.wadcfgx__. Using the properties box > Configuration > Enable Diagnostics > Configure. Give Storage Account Credentials to were diagnostics will be copied. 
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

#### Profile resource consumption
page 231.

#### Enable remote debugging

#### Establish a connection using Remote Desktop cmdlets in Windows PowerShell Azure PowerShell

#### Debug using IntelliTrace

### Debug using the Emulator