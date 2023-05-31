### ISOLATED

#### in .csproj file

##### only beta working. v1+ HttpTriggers stops working??
```xml
<PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.DurableTask" Version="0.4.1-beta" />
``` 

#### Create a normal HttpTrigger

##### Replace

```csharp

        [Function(nameof(StarterAsync))]
        public async Task<HttpResponseData> StarterAsync(
        [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequestData req,
        [DurableClient] DurableTaskClient client)
        {
            _logger.LogInformation("C# HTTP trigger function processed a request.");

            var instanceId = await client.ScheduleNewOrchestrationInstanceAsync(nameof(HelloCitiesOrchestrator));

            var response = client.CreateCheckStatusResponse(req,instanceId);
            return response;
        }
        [Function(nameof(HelloCitiesOrchestrator))]
        public async Task<string> HelloCitiesOrchestrator([OrchestrationTrigger] TaskOrchestrationContext context)
        {
            string result = "";
            result += await context.CallActivityAsync<string>(nameof(SayHelloActivity), "Auckland") + " ";
            result += await context.CallActivityAsync<string>(nameof(SayHelloActivity), "London") + " ";
            result += await context.CallActivityAsync<string>(nameof(SayHelloActivity), "Seattle");
            return result;
        }

        [Function(nameof(SayHelloActivity))]
        public async Task<string> SayHelloActivity([ActivityTrigger] string cityName)
        {
            _logger.LogInformation("Saying hello to {name}", cityName);
            await Task.Delay(15000);
            return $"Hello, {cityName}!";
        }


```


#### Call the HttpTrigger, get the **statusQueryGetUri**

Keep calling the *status Url* until completed (get output when runtimeStatus is *completed*)


#### Working

```xml

  <ItemGroup>
    <PackageReference Include="Microsoft.Azure.Functions.Worker" Version="1.10.0" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.DurableTask" Version="1.0.0" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.Http" Version="3.0.13" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Sdk" Version="1.7.0" OutputItemType="Analyzer" />
  </ItemGroup>

```

#### Not working

```xml

    <PackageReference Include="Microsoft.Azure.Functions.Worker" Version="1.8.0" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.Http" Version="3.0.13" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Sdk" Version="1.7.0" />
   <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.DurableTask" Version="1.0.2" />

```

#### Final

```xml

  <ItemGroup>
    <PackageReference Include="Microsoft.Azure.Functions.Worker" Version="1.10.0" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.Http" Version="3.0.13" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Sdk" Version="1.7.0" />
   <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.DurableTask" Version="1.0.2" />
  </ItemGroup>

```


#### Get instances

```powershell

/runtime/webhooks/durabletask/instances

```

#### Patterns

##### Sequentials (Chaining)

```csharp

[Function(nameof(HelloCitiesOrchestrator))]
        public async Task<string> HelloCitiesOrchestrator(
            [OrchestrationTrigger] TaskOrchestrationContext context)
        {
            string result = "";
            result += await context.CallActivityAsync<string>(nameof(SayHelloActivity), "Auckland") + " ";
            result += await context.CallActivityAsync<string>(nameof(SayHelloActivity), "London") + " ";
            result += await context.CallActivityAsync<string>(nameof(SayHelloActivity), "Seattle");
            return result;
        }


```

##### Fan out/Fan in


```csharp

       [Function(nameof(HelloCitiesOrchestrator))]
        public async Task<string> HelloCitiesOrchestrator(
            [OrchestrationTrigger] TaskOrchestrationContext context)
        {
      
            List<Task<string>> tasks = new();
            tasks.Add(context.CallActivityAsync<string>(nameof(SayHelloActivity), "Auckland"));
            tasks.Add(context.CallActivityAsync<string>(nameof(SayHelloActivity), "London"));
            tasks.Add(context.CallActivityAsync<string>(nameof(SayHelloActivity), "Seattle"));
            await Task.WhenAll(tasks);
            string result = string.Join(" ", tasks.Select(t => t.Result));
            return result;
        }


```

##### Human Approval/Interaction


```csharp

       [Function(nameof(HelloCitiesOrchestrator))]
        public async Task<string> HelloCitiesOrchestrator(
            [OrchestrationTrigger] TaskOrchestrationContext context)
        {
            using var timeoutCts = new CancellationTokenSource();
            DateTime dueTime = context.CurrentUtcDateTime.AddMinutes(3);
            Task durableTimeout = context.CreateTimer(dueTime, timeoutCts.Token);
            Task<bool> approvalEvent = context.WaitForExternalEvent<bool>("ApprovalEvent");
             if (approvalEvent == await Task.WhenAny(approvalEvent, durableTimeout))
            {
                timeoutCts.Cancel();
                //await context.CallActivityAsync("ProcessApproval", approvalEvent.Result);
                System.Console.WriteLine($"Back from approver: {approvalEvent.Result}");
            }
            else
            {
                //await context.CallActivityAsync("Escalate", null);
                System.Console.WriteLine("Timeout");
            }

            List<Task<string>> tasks = new();
            tasks.Add(context.CallActivityAsync<string>(nameof(SayHelloActivity), "Auckland"));
            tasks.Add(context.CallActivityAsync<string>(nameof(SayHelloActivity), "London"));
            tasks.Add(context.CallActivityAsync<string>(nameof(SayHelloActivity), "Seattle"));
            await Task.WhenAll(tasks);
            string result = string.Join(" ", tasks.Select(t => t.Result));
            return result;
        }



```

##### Give approval


```powershell

Clear-Host;

$baseUrl = "http://localhost:7071";
$instanceId = "dd6bc4c96bbb4004a5c492803d397f74";
$eventName = "ApprovalEvent";

$body = "false";

$url = "$baseUrl/runtime/webhooks/durabletask/instances/$instanceId/raiseEvent/$eventName";

$response = invoke-webrequest -uri $url `
		-Body $body `
		-Method Post `
		-Headers @{ "Content-Type" = "application/json" };
$response;




```
