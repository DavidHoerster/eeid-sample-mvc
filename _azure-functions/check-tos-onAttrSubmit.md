```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<object> Run(HttpRequest req, ILogger log)
{
    log.LogInformation($"C# HTTP trigger function processed a request.");

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();

    log.LogInformation($"Request... {requestBody}");
    
    dynamic request = JsonConvert.DeserializeObject(requestBody);

    // Parse specified attributes
    bool? isTouAccepted = request?.data?.userSignUpInfo?.attributes?.extension_8f501e8ef99546fe900eca1aa5a0c08f_isTouAccepted?.value;
    string displayName = request?.data?.userSignUpInfo?.attributes?.displayName?.value;
    string firstName = request?.data?.userSignUpInfo?.attributes?.givenName?.value;
    string lastName = request?.data?.userSignUpInfo?.attributes?.surname?.value;

    // JSON convert makes the length different, so we put 7 here
    if(!isTouAccepted.HasValue){
        var inputs = new Dictionary<string, string>();
        inputs.Add("extension_8f501e8ef99546fe900eca1aa5a0c08f_isTouAccepted", "You must accept terms of use");

        string pageMessage = "Please fix below errors to proceed";

        var actions = new List<ValidationErrorActions>{
            new ValidationErrorActions { type = "microsoft.graph.attributeCollectionSubmit.ShowValidationError", message = pageMessage, attributeErrors = inputs }
        };

        
        var dataObject = new Data {
            type = "microsoft.graph.onAttributeCollectionSubmitResponseData",
            actions= actions
        };

        dynamic response = new ResponseObject {
            data = dataObject
        };

        log.LogInformation($"Returning validation error - {dataObject}");

        // Send the validation error response
        return response;

    }else{
        var attrs = new Dictionary<string, object>(){
            {"extension_8f501e8ef99546fe900eca1aa5a0c08f_touVersion", "v19.0"},
            {"extension_8f501e8ef99546fe900eca1aa5a0c08f_isTouAccepted", isTouAccepted.Value},
            {"displayName", displayName},
            {"givenName", firstName},
            {"surname", lastName}
        };

        // var actions = new List<ContinueWithDefaultBehavior>{
        //     new ContinueWithDefaultBehavior { type = "microsoft.graph.attributeCollectionSubmit.ContinueWithDefaultBehavior"}
        //     };

        var actions = new List<ModifiedAttributesAction>{
            new ModifiedAttributesAction { 
                type = "microsoft.graph.attributeCollectionSubmit.modifyAttributeValues", 
                attributes = attrs 
            }
        };
        
        var dataObject = new DataModify {
            type = "microsoft.graph.onAttributeCollectionSubmitResponseData",
            actions= actions
            };

        dynamic response = new ResponseObjectModify {
            data = dataObject
        };

        log.LogInformation($"Returning continue");
        
        // Send the continue response
        return response;
    }
}

public class ResponseObject
{
    public Data data { get; set; }
}

[JsonObject]
public class Data {
    [JsonProperty("@odata.type")]
    public string type { get; set; }
    public List<ValidationErrorActions> actions { get; set; }
}

[JsonObject]
public class ValidationErrorActions {
    [JsonProperty("@odata.type")]
    public string type { get; set; }
    public string message { get; set; }
    public Dictionary<string, string> attributeErrors {get; set;}
}


public class ResponseObjectContinue
{
    public DataContinue data { get; set; }
}

public class ResponseObjectModify
{
    public DataModify data { get; set; }
}

[JsonObject]
public class DataContinue {
    [JsonProperty("@odata.type")]
    public string type { get; set; }
    public List<ContinueWithDefaultBehavior> actions { get; set; }
}

[JsonObject]
public class DataModify {
    [JsonProperty("@odata.type")]
    public string type { get; set; }
    public List<ModifiedAttributesAction> actions { get; set; }
}

[JsonObject]
public class ModifiedAttributesAction {
    [JsonProperty("@odata.type")]
    public string type { get; set; }
    public Dictionary<string, object> attributes { get; set; }
}

[JsonObject]
public class ContinueWithDefaultBehavior {
    [JsonProperty("@odata.type")]
	public string type { get; set; }
}
```