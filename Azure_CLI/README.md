### Login

```powershell

az login --use-device-code ## if needed, otherwise default browser

```

### View current subscription

```powershell

az account show

```

### View subscriptions

```powershell

az account list -o table

```

### Change subscription

```powershell

az account set -s [subscriptionId]

```