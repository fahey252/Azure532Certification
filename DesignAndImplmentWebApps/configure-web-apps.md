#Configure Web Apps
  * <https://azure.microsoft.com/en-us/documentation/articles/web-sites-configure/>

##Define and use app settings, connection strings, handlers, and virtual directories
  * App settings in portal take precedence over `web.config` values.  So development values are in source control and real values are not shared.  App settings are loaded when your application starts.
  * App setting and connection strings are separate in the portal but both key value pairs. Connection strings have a little extra metadata attached.
  * Stored as environment variables. Can access at run time two ways via:
  ```
  @Environment.GetEnvironmentVariable("APPSETTING_app-setting-key-name")
  // or
  @System.Configuration.ConfigurationManager.ConnectionStrings["name"]
  ```
  * App settings environment variable names are prefixed with **APPSETTING_** and conntection strings are prefixed with based on database type such as **(SQLAZURE|SQL|MYSQL|CUSTOM)CONNSTR_**
  * App settings can be set via powershell as well
  * 
  ```powershell
  Set-AzureWebsite <site-name> -AppSettings @{"key": "value"}
  ```
  * Use handlers to intercept HTTP requests of a certain type. i.e. jpegs. Below is the class and method signatures as well as some response psuedo-code
  ```
  public class JpegHandler1 : IHttpHandler
  public void ProcessRequest(HttpContext context)
  context.Response.Clear();
  context.Response.ContentType = "image/jpeg";
  context.Response.BinaryWrite(cloudBlobBytes);
  context.Response.End();
  ```
  * Register handlers in `<system.webServer>` config via an elment such as
  ```xml
  <handlers>
    <add name="JpegHandler" verb="*" path="*.jpg" type="JpegHandler" resourceType="Unspecified"/>
  </handlers>
  ```
  * Requests to files that match the file extension will be processed by the script processor/handler. Use the path `D:\home\site\wwwroot` to refer to your app's root directory.
  * Virtual directories take the path of your URL and map them to a location on the web server. i.e. example.com/drupal serves files /apps/drupal/drupal-7.30
  * To add a virtual directory, go to the Configure blade for the site and add a virtual path such as `/blog` for the key and path on the file system as the value. i.e. `site/wwwroot/drupal`. If the contents of the directory will be an application, such as web services, check the `application` checkbox as well.
  * When deploying, make sure you specify the virtual directory path so the services go into the virual directory.
  * Links
  	- <https://blogs.msdn.microsoft.com/subodhpatil/2013/08/02/importance-of-app-settings-in-azure-website-portal/>
  	- <https://blogs.msdn.microsoft.com/tomholl/2014/09/21/deploying-multiple-virtual-directories-to-a-single-azure-website>

##Configure certificates and custom domains

##Configure SSL bindings and runtime configurations

##Manage Web Apps by using the API, Azure PowerShell, and Xplat-CLI