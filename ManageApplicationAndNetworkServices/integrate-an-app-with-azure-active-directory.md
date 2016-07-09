### Integrate an app with Azure Active Directory
  * Azure Active Directory (Azure AD) provides a cloud-based __identity management service__ for application __authentication__, Single Sign-On (__SSO__), and __user management__ (Graph API). 
  * Manage some aspects of Azure AD with Windows PowerShell - initialize the deployment, create applications, __permissions, users, and groups__.

#### Develop apps that use WS-federation, OAuth, and SAML-P endpoints
  * __Create a directory__
    - Management Portal > New > App Services > Active Directory > Directory > Custom Create. Give it a name and `some-domain.onmicrosoft.com`. This is your tenant.
    - __Roles vs Groups: Roles - for AD functionality. Groups: Application authorization.__
  * __Manage users__
    - CRUD users from portal, Graph API, sync users from Windows Active Directory Server. __Azure Active Directory Connect (Azure AD Connect)__ previously DirSync and Azure AD Sync.
    - When creating a user, get an ID such as: user1@solexpaad.onmicrosoft.com
    - User will be asked to __change the password on first login__. You can optionally __email this password__ to the user’s email address.
    - Impractical to create all users. Usually scripted.
    - Azure AD makes it easy to __enable multi-factor authentication__ for users in your directory.
    - Pre-defined organizational roles for users: __(Global|Billing|Service|User|Password_ administrator__.
    - Azure AD roles are assigned only to users who have access to administer the Azure AD instance. __These roles are not used for application-specific authorization__.
    - You can optionally __create Azure AD groups to use for application authorization__of users in your directory; however, Azure AD roles are used to authorize access to Azure AD functionality. 
    - Azure AD __groups can not be used to authorize access to Azure AD resources__ such as user management and password resets. __Groups are for application purposes only!__
    - You can create groups for a directory and assign users to those groups. i.e. Guest, User, Owner.
    - Can assign a child group to a parent group to create a __hierarchy of groups__.
  * __Integrate applications__: 
    - Browser-based applications, call Web APIs from JavaScript, mobile applications that call Web APIs, server applications call Web APIs.
    - Choose from a few protocols: __WS-Federation (legacy), SAML-P (legacy), OAuth 2, or OpenID Connect__. Can be implemented using __OAuth 2.0 flows__, though this is not a strict requirement.  Azure AD exposes endpoints for WS-Federation metadata and sign-on endpoints, SAML-P sign-on and sign-out endpoints, OAuth 2.0 token and authorization endpoints, Azure AD Graph API endpoints.
      + https://login.windows.net/4a3779e5-8c00-478d-b2cd-7be06a8b236e/(wsfed|saml2|oauth2) - or urls similar to that.
      + You can substitute the tenant identifier with the tenant name for protocol endpoint URLs. i.e. <https://login.windows.net/fahey.onmicrosoft.com/wsfed>.
    - __OpenID Connect is recommended__ path because it is the most modern protocol available.
    - Create your application. Register with Azure AD. Write code in your application to satisfy one of the scenarios for user authentication or token requests to call APIs. Receive protocol-specific responses to your application from Azure AD, including a __valid token for proof of authentication or for authorization purposes__.
      + For __Sign-On URL__, provide the URL to your application. Used as the __reply URL__ for browser-based scenarios so that the response from user authentication can be sent to your application. You should provide the __URL that will process this response__ for this value.
      + __App ID URI__, provide a unique URL that will identify your application. This does not have to be a live endpoint. __Must be unique across all applications in the Azure AD tenant.__
      + Settings: Logo, single or multi-tenant, OAuth keys, permissions.
    - __WS-Federation__ is an identity protocol used for browser-based applications.
      + User is presented with a login page, unless they __already has a valid cookie__ for the Azure AD tenant.
      + __SAML token is returned__ in the HTTP POST to the application URL with a WS-Federation response (reply URL).
      + The application processes this response, __verifies the token is signed__ by a trusted issuer (Azure AD), and confirms that the __token is still valid.__
      + Can optionally __use claims in the token to personalize__ the application experience for the logged in user.
      + Can optionally __query Azure AD for groups__ for authorization purposes.
      + __Request a user's groups post-authentication by sending a request to the Graph API.__
      + __WS-Federation exposes two endpoints__, one for __metadata__ and one for sign-in and sign-out.
      + Metadata endpoint exposes the standard federation metadata document that many identity tools know how to consume.
      + Can use the OWIN framework to handle WS-Federation/OpenID requests and responses.
    - __SAML 2.0 Protocol (SAML-P) can be used like WS-Federation__ to support user authentication to browser-based applications.
      + SAML-P tools are not provided as part of the .NET Framework libraries. SAML-P becomes important when you are integrating other SaaS applications with your Azure AD because __some applications do not support WS-Federation or OpenID Connect__.
    - __OAuth 2.0__ is an authorization protocol and is __not used for classic browser-based authentication__ scenarios. OpenID Connect (OIDC) is an identity protocol that __extends OAuth 2.0__ to provide a mechanism for authenticating users.
      + request that indicates the __application URI for the client_id__ element in the request.
      + Reply URL, and this value should match the redirect_uri element in the request. The __response contains a JSON Web Token (JWT)__. Verify token, personalize, groups.
      + Azure AD exposes two OAuth endpoints: one for __token requests and one for authorization requests__ typical of OAuth 2.0.
    - OAuth 2.0 is typically useful in __scenarios where user consent is required to access resources__, but it can also be employed to request tokens for server side client applications. In both cases, your application __must supply a client identifier and a key (secret) when requesting a token__ from the OAuth token endpoint.
      + Select a duration of __one or two years for the key__ to be valid.
      + Copy the key somewhere safe; it will not be presented again.
      + OAuth requests can be __made for users or for applications__.
  * __OAuth is an authorization protocol, not an authentication protocol__. You can use WS-Federation, SAML-P, or OpenID Connect (an extension to OAuth) as browser-based authentication protocols.
  * OpenID Connect requires that the application is trusted by identifying the application URI provided in the request for user authentication, but the __client identifier is not used for OpenID Connect flow__. The client identifier is required for OAuth requests, and, in cases where the user is not authenticating, the client key (secret) is also required.
  * Links
    - [Authentication Scenarios for Azure AD](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/)
    - [Azure AD Code Samples](https://azure.microsoft.com/en-us/documentation/samples/?service=active-directory)
  
#### Query the directory by using graph API
  * Using the Graph API, you can interact with your Azure AD tenant to manage users, groups, and more. Applications are __given permissions__ (read/write) to interact with AD. Find users, Request a user’s group and role membership, Manage group membership, __Create applications, Query and create directory properties__.
  * Before you can interact with the Graph API programmatically, you __must grant the calling application access to Graph API__.
    - Create Application in AD > Persmissions > select __Read Directory Data__ or Read And Write Directory Data permissions.
  * Using the __Active Directory Client Library__, you can write code to securely reference the Graph API for your Azure AD tenan.
  * Use the following NuGet packages: __Active Directory Authentication Library Microsoft Azure Active Directory Graph Client Library__
  
  ```c#
  string graphUrl = "https://graph.windows.net";
  string tenantId = "4a3779e5-008c-478d-cdb2-7be06a8b236e";
  string loginUrl = "https://login.windows.net/solexpaad.onmicrosoft.com";
  string clientId = "dafe0335-446b-9845-8653-6f920b0623dc";
  string clientSecret = "Xha6cdPlVdABiz+StfSZZBwP4eWntuxokLtSPSaiOFg=";

  using Microsoft.Azure.ActiveDirectory.GraphClient;
  string serviceRoot = graphUrl + "/" + tenantId;
  var graphClient = new ActiveDirectoryClient(new Uri(serviceRoot), async () => await GetTokenAsync());

  var authContext = new AuthenticationContext(loginUrl, false);
  var creds = new ClientCredential(clientId, clientSecret);
  Task<AuthenticationResult> result = authContext.AcquireTokenAsync(graphUrl,
  creds);
  string token = result.Result.AccessToken;

  string upn = "admin@solexpaad.onmicrosoft.com";
  List<IUser> matchingUsers = activeDirectoryClient.Users
    .Where(user => user.UserPrincipalName.Equals(upn))
    .ExecuteAsync().Result.CurrentPage.ToList();
  var foundUser = (User) matchingUsers.First();
  ```
  * You will __always need a way to retrieve groups__ and other user profile information after they have authenticated.
  * Roles are defined by Azure AD for administering specific directory features; groups are customizable for the application and are typically assigned to users for authorization. __Users cannot be assigned to roles via Graph API.__
  * Links
    - [Integrating applications with Azure Active Directory](https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/#BKMK_Graph)


