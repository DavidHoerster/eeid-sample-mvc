```csharp
#r "Newtonsoft.Json"
using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;
public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");
    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);

    // Read the correlation ID from the Azure AD  request    
    string correlationId = data?.data.authenticationContext.correlationId;

    var random = new System.Random();
    var loyaltyValues = (LoyaltyLevel[])Enum.GetValues(typeof(LoyaltyLevel));
    // Generate a random index to select a LoyaltyLevel
    int randomIndex = random.Next(loyaltyValues.Length);
    // Assign the randomly selected LoyaltyLevel to the "claim" variable
    LoyaltyLevel claim = loyaltyValues[randomIndex];

    // Claims to return to Azure AD
    ResponseContent r = new ResponseContent();
    r.data.actions[0].claims.CorrelationId = correlationId;
    r.data.actions[0].claims.ApiVersion = "1.0.0";
    r.data.actions[0].claims.DateOfBirth = "01/01/2000";
    r.data.actions[0].claims.Level = claim.ToString();
    return new OkObjectResult(r);
}

public class ResponseContent{
    [JsonProperty("data")]
    public Data data { get; set; }
    public ResponseContent()
    {
        data = new Data();
    }
}

public class Data{
    [JsonProperty("@odata.type")]
    public string odatatype { get; set; }
    public List<Action> actions { get; set; }
    public Data()
    {
        odatatype = "microsoft.graph.onTokenIssuanceStartResponseData";
        actions = new List<Action>();
        actions.Add(new Action());
    }
}

public class Action{
    [JsonProperty("@odata.type")]
    public string odatatype { get; set; }
    public Claims claims { get; set; }
    public Action()
    {
        odatatype = "microsoft.graph.tokenIssuanceStart.provideClaimsForToken";
        claims = new Claims();
    }
}

public enum LoyaltyLevel {
    Bronze,
    Silver,
    Gold,
    Platinum,
    Adamantium,
    Vibranium
}

public class Claims{
    [JsonProperty(NullValueHandling = NullValueHandling.Ignore)]
    public string CorrelationId { get; set; }
    [JsonProperty(NullValueHandling = NullValueHandling.Ignore)]
    public string DateOfBirth { get; set; }
    public string ApiVersion { get; set; }
    public string Level { get; set; }
    public Claims()
    {
    }
}
```