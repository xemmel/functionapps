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
        [DurableClient] DurableClientContext durableContext)
        {
            string instanceId = await durableContext.Client.ScheduleNewOrchestrationInstanceAsync(nameof(HelloCitiesOrchestrator));
            _logger.LogInformation("Created new orchestration with instance ID = {instanceId}", instanceId);

            return durableContext.CreateCheckStatusResponse(req, instanceId);
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