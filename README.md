# CORS sample ASP.NET Core on .NET 6 and IIS 10 on Azure AppService

## What this demo project shows

This sample shows, based on the `dotnet new webapi` sample project, how to enable CORS in your ASP.NET Core WebApi. It uses the most basic approach with a default policy that will be applied to all controllers without any further attributes.

## Changes required to enable CORS in the ASP.NET Core project

All required changes are in the `Program.cs` file:

### Define a default policy

```csharp
// Add services to the container.
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(builder =>
        {
            builder.WithOrigins("http://example.com");
        });
});
```

### Use CORS

```csharp
app.UseCors();
```

That's it. You can work from here to use named policies if you want more granular control.

## Deploy to IIS or Azure AppService (Windows)

If you deploy a .NET 6 project to IIS (on premise or via Azure AppService for Windows), there might arise issues as browsers might use preflight requests with the HTTP `OPTIONS` method to check for CORS. By default, IIS does not forward this request to the ASP.NET Core runtime, instead it gets handled by the `OPTIONSVerbHandler` handler.

### Remove the OPTIONS handler

To mitigate that, it's as easy as to remove this handler in the `<system.webServer>` section of `web.config`.

Do this by adding `<remove name="OPTIONSVerbHandler" />` on top of the `<handlers>` section:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <remove name="OPTIONSVerbHandler" />
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet" arguments=".\repos.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

What happens is that now, the `aspNetCore` handler below will receive the `OPTIONS` requests and can handle them as `verb` is defined to handle all HTTP verbs, including `OPTIONS`.
