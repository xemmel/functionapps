Create new Function App (new RG)
Create new Key Vault -> NEXT
    -> Yourself (Key Vault Admin)

Create a secret in Key vault
Function App
     Function Http


```csharp

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");
        string password = Environment.GetEnvironmentVariable("password");

            return new OkObjectResult($"The password is: {password}");
}

```

App Setting (password)

@Microsoft.KeyVault(VaultName=...;SecretName=...)