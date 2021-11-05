# Synigo Pulse One API

Synigo Pulse One API is a C#(5) solution, containing projects which you can use to easily create API's using the latest Microsoft (Azure) standards as it comes to principles and hosting.

The goal of this solution to let you as a developer create API's quickly, without worrying about any of the boiler plate code out there. We try to achieve this by:

- Generating Nuget packages out of this code, which you can use in your projects.
- Creating abstractions based on both **ASP.NET Core (5)** and **Azure Functions**
  - With a minimum of code, you can easily setup a solution and profit from everything in there.
  - We try to prepare everything for you and allow you to extend or modify behavior if needed.   


## ASPNET Core

Follow these instructions if you want to implement a ASP.NET Core solution:

## Step 1 pull in our Nuget Packages



| Package                       | Description                                                  |
| ----------------------------- | ------------------------------------------------------------ |
| **Synigo.OneAPI.Model**       | If you want to use our models which are designed to be read by our product Synigo Pulse com, get this one! |
| **Synigo.OneAPI.Core**        | Contains some of the core functionality used by the other packages |
| **Synigo.OneAPI.Interfaces**  | Package Contains the interfaces we've implemented, we've created a seperate package for this one, if you want to create your own implementations |
| **Synigo.OneAPI.Providers**   | This package contains some implementations to some generic providers such as Redis for caching |
| **Synigo.OneAPI.Common**      | Shared methods and common stuff (we all have this right?)    |
| **Synigo.OneAPI.Core.WebApi** | If you have an ASPNET Core implementation (Web API) Pull in this package, it contains most of the code described below |
| **Synigo.OpenEducationAPI**   | This package contains a C# implementation of the V4 implementation of the Open Education API (https://open-education-api.github.io/specification/) |

*(Goto https://www.nuget.org/packages?q=synigo to see all of our Nuget packages )*



### Step 2 Inherit from  Synigo.OneApi.Core.WebApi.BaseStartup 

```c#
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Swashbuckle.AspNetCore.SwaggerGen;
using Synigo.OneApi.Core.WebApi;

public class Startup : BaseStartup // inherit this one
{
    public Startup(IConfiguration configuration) : base(configuration)
    {
    
    }
    // If you want to add Swagger documentation to your API
    // You can override ConfigureSwaggerGen and and configure it here...
    protected override void ConfigureSwaggerGen(SwaggerGenOptions options)
    {
        base.ConfigureSwaggerGen(options);
        options.EnableAnnotations();
    }
    
    // If you have any other Dependencies to register, you can override 
    // This method and add them here
    protected override void ConfigureCustomServices(OneApiBuilder builder)
    {
       
    }
}
```

## Step 3 Implement an API controller

```c#
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Synigo.OneApi.Core.WebApi;
using Synigo.OneApi.Interfaces;

[ApiController]
[Route("[controller]")]
public class MyController : OneApiController // inherit this one 
{
  // IExecutionContext gets injected here and is accesible using base.Context
  public MyController(IExecutionContext context) : base(context)
  {

	}
	
	  // This annotation will create a nice description for your Open Api Documentation
  [SwaggerOperation(
    Summary = "NICE SUMMARY OF YOUR ENDPOINT",
    Description = "MAYBE A DESCRIPTION?",
    OperationId = nameof(GetOffersAsync),
    Tags = new[] { "offer", "recruitee", "mylist" }
  )]
  [HttpGet("MYROUTE")]
  public async Task<IActionResult> GeMyStuffAsync() // implement this one
  {
    // Check if the user is authorized to use this endpoint
    // based on checking the JWT token for valid scopes
    var authResult = await (Context.Current.IsAuthorizedAsync("Api.Read.All", "Api.myspecialAction"));

    // Nope... not valid :-(
    if (!authResult.IsValid)
    {
      return new UnauthorizedResult();
    }
    // Return a value from an injected provider
    // Context gives you all kinds of relevant references to 
    // information regarding your current request
    return new OkObjectResult(_injectedProvider.GetSomeValue(Context.Current.Principal));  
  }
}

```

## Step 4 Setup your configuration

Below you'll find an example about how to connect to your Azure AD and needed to validate the request (JWT token) and maybe connect to your AD and do stuff...

See these links for more information:

- https://docs.microsoft.com/nl-nl/azure/active-directory/develop/scenario-token-exchange-saml-oauth 
- https://docs.microsoft.com/nl-nl/azure/active-directory/develop/msal-overview
- [https://jwt.io](https://jwt.io/) 

```json
{
    "AzureAd": {
        "Instance": "https://login.microsoftonline.com",
        "ClientId": "YOUR CLIENTID",
        "TenantId": "YOUR TENANTID",
        "ClientSecret": "YOUR CLIENTSECRET",
        "CallbackPath": "A CALLBACK PATH"
    },
    "Logging": {
        "LogLevel": {
            "Default": "Information",
            "Microsoft": "Warning",
            "Microsoft.Hosting.Lifetime": "Information"
        }
    },
    "AllowedHosts": "*"
}

```



