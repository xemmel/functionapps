
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

