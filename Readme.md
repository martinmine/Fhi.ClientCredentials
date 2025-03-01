﻿## Client Credentials Usage


## Configuration file section

1. Add the following configuration section to your appsettings.json files, and populate it appropriately.


```json
  "ClientCredentialsConfiguration": {
    "clientName": "",
    "authority": "",
    "clientId": "",
    "grantTypes": [ "client_credentials" ],
    "scopes": [ ],
    "secretType": "private_key_jwt:RsaPrivateKeyJwtSecret",
    "rsaPrivateKey": "",
    "rsaKeySizeBits": 4096,
    "privateJwk": "",
    "Apis": [
      {
        "Name": "", // Tip:  Use nameof(YourService)
        "Url": ""
      }
    ],
    "refreshTokenAfterMinutes":  8  // Set approx 20% less than lifetime of access token
  }
```



## Client Credentials using Keypairs

1. Add package 'Fhi.ClientCredentialsKeypairs' to your project

2. In your `Program.cs` file, or if older `Startup.cs`, add the following code section (for the outgoing interfaces):

```cs
    var clientCredentialsConfiguration = services.AddClientCredentialsKeypairs(Configuration);
    services.AddHttpClient(nameof(YourService), c =>
    {
       c.Timeout = new TimeSpan(0, 0, 0, 10);
       c.BaseAddress = clientCredentialsConfiguration.UriToApiByName(nameof(YourService));
    })
    .AddHttpMessageHandler<HttpAuthHandler>()
    .AddTypedClient(c => RestService.For<IExternalApi>(c, new RefitSettings
    {
       ContentSerializer = new SystemTextJsonContentSerializer(services.DefaultJsonSerializationOptions())
    }));
```
replacing `YourService` with the service you have done for accessing the external api, and replace `IExternalApi` with the Refit interface for whatever external api you want to access.

The `Configuration` property is the injected IConfiguration property from the Startup.cs file.

If you don't use Refit, you can just skip the last part, and get the named client from the injected HttpFactory in your service. It will still have the authenticationhandler, so you don't need to do anything more there to get the bearer token. It will be added automatically.




## Client Credentials using Client Secrets

If you want to disable the authorization for some reason, you can add another property named `Enable` to the ClientCredentialsConfiguration, it is default true.

2. Add package `Fhi.ClientCredentialsUsingSecrets` to your project
3. In your `Program.cs` file, create an instance of the `ClientCredentialsSetup` class using an `IConfiguration` parameter.
4. Using the created instance call the method `ConfigureServices`.

