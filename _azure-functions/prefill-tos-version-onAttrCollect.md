```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<object> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();

    log.LogInformation($"Request --  {requestBody}");
    
    dynamic request = JsonConvert.DeserializeObject(requestBody);

    // The form fields will load with these default values
    var inputs = new Dictionary<string, object>()
    {
        { "extension_<externsion-app-id>_touVersion", "v19.0" }      
    };

    var actions = new List<SetPrefillValuesAction>{
        new SetPrefillValuesAction { 
            type = "microsoft.graph.attributeCollectionStart.setPrefillValues", 
            inputs = inputs }
    };

    var dataObject = new Data {
        type = "microsoft.graph.onAttributeCollectionStartResponseData",
        actions= actions
    };

    dynamic response = new ResponseObject {
        data = dataObject
    };

    // Send the response
    return response;
}

public class ResponseObject
{
    public Data data { get; set; }
}

[JsonObject]
public class Data {
    [JsonProperty("@odata.type")]
    public string type { get; set; }
    public List<SetPrefillValuesAction> actions { get; set; }
}

[JsonObject]
public class SetPrefillValuesAction {
    [JsonProperty("@odata.type")]
    public string type { get; set; }
    public Dictionary<string, object> inputs { get; set; }
}
```