
### Getting started

#### Install Chocolatey (in powershell as administrator)

```powershell


Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

```

#### Install powershell core (7)

```powershell

choco install powershell-core -y

```

#### Install tools (In powershell 7 as admin)

```powershell


## choco install az.powershell -y
choco install azure-cli -y
choco install azure-functions-core-tools -y
choco install vscode -y
choco install postman -y
## choco install notepadplusplus -y
choco install git -y
choco install dotnet-sdk -y

```


### Logon to Azure (xx = 01,02...) (number given by instructor)

UserName: studentfunction05023xx@integration-it.com
Password: [given by instructor]

Do NOT set up MFA (Skip for now, etc.)


### Azure (Portal)

portal.azure.com

### Azure AD 

entra.microsoft.com



### Create Function Apps

#### In-process

```powershell

func init [name] --worker-runtime dotnet

```


#### Isolated

```powershell

func init [name] --worker-runtime dotnetIsolated --target-framework net7.0

```

#### If nuget.org not registered

```powershell

dotnet nuget add source https://api.nuget.org/v3/index.json -n nuget.org

```

#### Create new HttpTrigger (In project folder)

```

func new -t HttpTrigger -n [name]

```

#### Create new QueueTrigger (In project folder)

```

func new -t QueueTrigger -n [name]

```


#### Queue trigger code

```csharp

        [Function("MyQueueTrigger")]
        public async Task Run([QueueTrigger("invoicequeue", Connection = "theconnection")] string myQueueItem)
        {
            _logger.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
        }

```

#### Queue trigger local.settings.json

Find your storage connection string under *Keys*

```json

        "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
        "theconnection" : "....."
    }

```

#### Start the function app locally

```powershell

func start

```

#### Link to Storage Queue docs

https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue-output?tabs=python-v2%2Cisolated-process%2Cextensionv5&pivots=programming-language-csharp#example






##### Deploy Function App from CLI

Create new Function App (.NET 7 Isolated)

```powershell

func azure functionapp publish [name of function app]

```