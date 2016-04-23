#Configure Web Apps
  * <https://azure.microsoft.com/en-us/documentation/articles/web-sites-configure/>

##Define and use app settings, connection strings, handlers, and virtual directories
  * App settings in portal take precedence over `web.config` values.  So development values are in source control and real values are not shared.
  * App setting and connection strings are separate in the portal but both key value pairs. Connection strings have a little extra metadata attached.
  * Stored as environment variables. Can access at run time two ways via:
  ```
  @Environment.GetEnvironmentVariable("APPSETTING_app-setting-key-name")
  // or
  @System.Configuration.ConfigurationManager.ConnectionStrings["name"]
  ```

  * App settings environment variable names are prefixed with **APPSETTING_** and conntection strings are prefixed with based on database type such as **SQLAZURECONNSTR_**
  * App settings can be set via powershell as well
  * 
  ```powershell
  Set-AzureWebsite site-name -AppSettings @{"key": "value"}
  ```

  * Use handlers to intercept HTTP requests of a certain type. i.e. jpegs. Below is the class and method signatures as well as some response psuedo-code
  ```c#
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
  * Virtual directories take the path of your URL and map them to a location on the web server. i.e. `example.com/`drupal serves files `/apps/drupal/drupal-7.30`
  * To add a virtual directory, go to the Configure blade for the site and add a virtual path such as `/blog` for the key and path on the file system as the value. i.e. `site/wwwroot/drupal`. If the contents of the directory will be an application, such as web services, check the `application` checkbox as well.
  * When deploying, make sure you specify the virtual directory path so the services go into the virual directory.
  * Links
  	- <https://blogs.msdn.microsoft.com/subodhpatil/2013/08/02/importance-of-app-settings-in-azure-website-portal/>
  	- <https://blogs.msdn.microsoft.com/tomholl/2014/09/21/deploying-multiple-virtual-directories-to-a-single-azure-website>

##Configure certificates and custom domains
  * Must first add a custom domain before configuring for HTTPS (not available in the free pricing tier). Can buy one directly in the Azure Portal or use another domain registar.
  * Get a domain, create a DNS record that maps the domain to the Azure App Service then add the domain in the Portal.  Can map the root domain, a subdomains or a wildcard to Azure.
  * DNS has A Records (hostname to ip address) and CNAME's (domain mapped to another domain and uses the desitation domain to look up the IP address.).  Can use Azure DNS to host your DNS records for the app.
  * If you create an A record to a root domain or subdomain, you will also need to create a CNAME for __awverify__ to __ awverify.\<yourwebappname\>.azurewebsites.net__ to so Azure can validate you own the domain.
  * Cannot use CNAMEs in Azure until they have propagated.  Can not use A Records until *awverify* CNAME has been completed and verified.  Can check status of DNS propagation with <http://digwebinterface.com/>
  * To add the custom domain to your App Service Portal > App > Settings > Custom Domains and SSL > __Bring External Doamins__ > Then enter the list of domains to associate with the app > Save.
  * Custom domains can also be associated with Traffic Manager via *.trafficmanager.net.
  * To get a certificate, upgrade to __stanard/premium pricing tier to use SSL with a custom domain__.  Certificates can be for a single domain, multiple specified domains (SNI) or via wildcards(* for multiple subdomains). Based on the Subject Alternative Name (SAN) field in the certifcate.
  * Certificate must be from a CA (certificate authority), have a private key, exportable to `.pfx` file (Personal Information Exchange), cert subject name match domain name, minimum 2048-bit encryption.  Can use self-signed certs for testing only.
  * Will need to __submit a Certificate Signing Request (CSR)__. Then generate a `.pfx` from the certificate recieved.
  * Create a certificate with Certreq.exe (creates CSR's), IIS Manager or OpenSSL.
  * For certificates with multipe domains/wild card domains, will need a `subjectAltName` certificate.
  * To use certreq.exe, create a text file with the following/additonal configs
  ```
  [NewRequest]
  Subject = "CN=mysite.com"
  Exportable = TRUE
  KeyLength = 2048
  ```

  * Then run the text file through certreq.exe
  ```
  certreq -new \path\to\myrequest.txt \path\to\create\myrequest.csr
  ```

  * Once CSR is submitted to the CA, you will be given a .cer file.  To accept/complete the signature of the certificate, run
  ```
  certreq -new \path\to\myrequest.txt \path\to\create\myrequest.csr
  ```

  * Install the certicate on your machine by clicking on it and running the __Certificate Import Wizard__.  You can __export the certificate__ using `certmgr.msc`.  Export as .pfx file including all certificates, settings.  You can then __upload the .pfx to your app in the Azure App Service__. Be sure to __include the private key__ in the export.
  * To use OpenSSL, the process is pretty much the same, but use the commands
  ```bash
  $ openssl req -new -nodes -keyout myserver.key -out server.csr -newkey rsa:2048  # complete the prompts
  ```

  * You will then have the .key and .csr file to submit to the CA.  The CA will give you a .csr file.
  * Azure App Service only accepts .pfx files (__pkcs12__ when exporting (Public-Key Cryptography Standard by RSA)) so use the key file and certificate to generate a .pfx file. Give the .pfx file a password when prompted to secure it.
  ```bash
  $ openssl pkcs12 -export -out myserver.pfx -inkey myserver.key -in myserver.crt
  ```

  * __Note: must install all intermidate certificates before exporting the certificate__ so they are included in the export process.  To include all intermidate certificates, use the command
  ```bash
  $ openssl pkcs12 -chain -export -out myserver.pfx -inkey myserver.key -in myserver.crt -certfile intermediate-cets.pem
  ```

  * Uses the __SubjectAltName__ extension to support multiple domain names with a single certificate, however it requires a configuration file (i.e. sancert.cnf) for additonal keys.  subjectAltName key has comma delimited list of domains to include.  Use the command to include multiple domain configuration for the SSL certificate.
  ```bash
  $ openssl req -new -nodes -keyout myserver.key -out server.csr -newkey rsa:2048 -config sancert.cnf
  ```

  * Can also use IIS Manager to generate the CSR and contact a CA.
  * Can use self signed certs for non-production environments using makecert.exe or OpenSSL. Browsers will warn may not be secure.  The self-signed cert can be exported via as .pfx file using the key file and .cert file, then upload to the Azure app.
  * To configure your app for SSL in a Standard/Priemium tier, go to App Settings > Routing > Custom Domains/SSL > Certficates > Upload. Use the .pfx file generated and password used when exporting.
  * Setup the SSL bindings to use per domain and which certificate to use. Can use Server Name Indication (SNI) or IP based SSL.
    - __IP based__: when hostname has dedicated public IP address.  This is the traditional method.  May need to updated the DNS A record where domain is hosted to point to the virtual IP address your domain is hosted on in Azure.
    - __Server Name Indication__: Extension of SSL and TLS that allows multiple domains to share the same IP address/how certificates use seperate certs for each domain.  Most browsers support SNI.
  * __Azure does not enforce HTTPS__. users may still access the HTTP version of the app.  Use the URL Rewrite module to force HTTPS. .NET MVC applications should use the __RequireHttps filter__ instead of URL Rewrite.
  * Can add the rewrite rule in the web.cofig file of the application. Cause a 301 redirect.
  ```xml
  <configuration>
    <system.webServer>
      <rewrite>
        <rules>
          <rule name="Force HTTPS" enabled="true">
            <match url="(.*)" ignoreCase="false" />
            <conditions>
              <add input="{HTTPS}" pattern="off" />
            </conditions>
            <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" appendQueryString="true" redirectType="Permanent" />
          </rule>
        </rules>
      </rewrite>
    </system.webServer>
  </configuration>
  ```

  * Note: __Node.js, Python Django, and Java all actually do use a web.config when hosted on Azure App Service__. Azure creates the file automatically during deployment, so you never see it. If you include one as part of your application, it will override the one that Azure automatically generates.  Can get the file by downloading via FTP.
  * Links
    - [Configure a custom domain name in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/web-sites-custom-domain-name/)
    - [Enable HTTPS for an app in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/web-sites-configure-ssl-certificate/)

##Configure SSL bindings and runtime configurations

##Manage Web Apps by using the API, Azure PowerShell, and Xplat-CLI