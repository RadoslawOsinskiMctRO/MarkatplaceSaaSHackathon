## Challenge 4: Implement WebHook

In this challenge you will dive deeper in webhook implementation.
You will add a new functionality to notify you in case if the subscriber will cancel the subscription from the Azure portal. It will be done through adding this functionality to the WebHook. 

### Pre-requisites

You have to have the dotnet and PowerShell installed. Visual Studio is optional. You have to have a deployed customer portal form saas accelerator.

### What is the WebHook and what we can and should implement there

The webhook is where Microsoft will inform you about changes — when you’ll have to reconfigure your user’s account on plan changes, to disable it when the subscription ends, or payment fails. WebHook is working on your side and you as a publisher provide the URL for this webhook endpoint when registering the offer for Azure Marketplace.

It has to be available 24/7, and Microsoft will keep trying to call it until you return 200. Webhook is a mechanism to drive the actual SaaS subscription activation, usage, updates, and cancellation so it needs to work seamleassly.

The list of operations that are placed there include:
When the SaaS subscription is in Subscribed status:
ChangePlan
ChangeQuantity
Suspend
Unsubscribe

When SaaS subscription is in Suspended status:
Reinstate
Unsubscribe

Change plan, change quantity, and unsubscribe actions are tested from the publisher side. From the Microsoft side, unsubscribe can be triggered from both the Azure portal and Admin Center (the portal where Microsoft AppSource purchases are managed). Change quantity and plan can be triggered only from Admin Center.

### WebHook Sample Implementation and Workflow

Your deployed sample WebHook implementation for CustomerPortal is put into the src/SaaS.SDK.CustomerProvisioning folder and consist of 2 files in 2 sub-folders.

Controllers/WebHook/AzureWebHookController
WebHook/WebHookHandler

WebHookPayload sets the API Response payload model.

Payload package goes to the WebHookProcessor that looks into the payload.Action field and executes the corresponding action instantiated by WebHookAction class from the WebHookHandler class.

After an update is entered, Microsoft will call the publisher's webhook URL, configured in the Connection webhook field on the Technical configuration page in Partner Center, with an appropriate value for action and other relevant parameters.

AzureWebHookController is a code that wraps the WebHook functionality and thus should be used to any functionality that is not core to the webhook.

![scheme](https://docs.microsoft.com/en-us/azure/marketplace/partner-center-portal/media/saas-update-status-api-v2-calls-marketplace-side.png)

### Setting up the WebHook interceptor

You want to receive a notification when a customer is cancelling the subscription using Azure portal. The simplest way to implement this is to add the interceptor into the WebHook. The only downside is that you will need to do it as careful as possible as if the code will have any defects, the WebHook will start to throw errors.

In our case, for this event, we set up for you the Azure Logic App for emailing, and we will need to add an HTTP call to our WebHook that will be executed each time when someone is cancelling the subscription. This code does not have any error processing logic, should not be used in production and used only in a purpose of demo.

#### Coding

There are two places where you can put the code, although it will be a little different.
It can be put into WebhookHandler.cs and it will work, however this will go against best practices about how we divide the functionality. The WebHookHandler class mission is to provide a functionality to handle different types of request. Intercepting the request here can be a good idea, however if the interceptor code will fail, we will likely lose the request - even with the retry logic Microsoft is using. 

Instead we add this to the AzureWebhookController.cs which is the controller where we handle the high-level logic of handling the request.

Open AzureWebhookController.cs.

Add imports to the very beginning. We will use few useful standard classes.

```c#
using System.Text;
using System.Net.Http;
using Newtonsoft.Json; 
```

Right after this line, paste the Customer class definition. This is where we are defining the request schema that will represent the customer request and will go to our web-service.

```c#
public class Customer
{
    public string Customer;
    public string Email;

    public Customer(string custName, string adminEmail)
    {        
            Customer = custName;
            Email = adminEmail;
    }
} 
```


Now, look for this line

```c#
public async void Post(WebhookPayload request)
```


Replace the content of this method starting with "if" clause. Replace the "INSERT YOUR EMAIL" with your email - this is where the notification will be sent.

Explanation of the code:
* If the request is properly formatted and has values, continues with the execution
* Webhook execution is logged
* If the Action is Unsubscribe (you can look at the WebHook actions for more actions if you will want to intercept other operations), proceed with the notification
* url is the HTTP URL of the Logic App that we created that will receive the customer id and your email and send the email using O365 
* HTTP Client is instantiated
* Customer is the class that we use to model the request and then serialize it into JSON
* We make a POST request

```c#
if (request != null)
   {
     var json = System.Text.Json.JsonSerializer.Serialize(request);
     this.applicationLogService.AddApplicationLog("Webhook Serialize Object " + json);
     await this.webhookProcessor.ProcessWebhookNotificationAsync(request).ConfigureAwait(false);
     if (request.Action == WebhookAction.Unsubscribe) { 
     var url = "https://prod-23.northcentralus.logic.azure.com:443/workflows/6a7d44b2538943488a153a882ca3caec/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=jV-Sw8BAI8kxt0-zjXmPxTF3s7u-SIdtYqWhkaCf8b8";
     
     using var client = new HttpClient();
     Customer cust = new Customer ("your customer "+subscriptionsRepository.GetById(request.SubscriptionId).PurchaserEmail+" decided to cancel the subscription", "[INSERT YOUR EMAIL HERE]");
            
     var jsoncust = JsonConvert.SerializeObject(cust);
     var data = new StringContent(jsoncust, Encoding.UTF8, "application/json");

     var response = await client.PostAsync(url, data);
                 }
}
```

Re-build (CTRL+B) and re-deploy the project.
For this, if you can't deploy it using VS for any reason, you can use command line.

Open Powershell.

Go to the (cd [directory]) to the CustomerProvisioning directory.

```
\Commercial-Marketplace-SaaS-Accelerator\src\SaaS.SDK.CustomerProvisioning\
```

Execute this command.

```powershell
dotnet publish .\SaaS.SDK.CustomerProvisioning.csproj -c debug -o CustomerPortal
```

Execute this command.

```powershell
Compress-Archive -Path .\CustomerPortal\* -DestinationPath CustomerPortal.zip -Force
```

This will prepare our project to be deployed.

Go to Kudu website. Change the "link" to the value of your website, DO NOT CHANGE anything else.
```
https://yourwebsite.scm.azurewebsites.net/ZipDeployUI
```
Drag and drop the CustomerPortal.zip to the UI with the directory listing. This will cause the upload and deployment.

Now, subscribe to the offer, activate it using your Admin portal and cancel the subscription from the portal. You should see email coming to your mailbox with the subscription ID that you can use to identify the customer and contact him if needed.

## Success Criteria
1. You will receive notification on email when you cancel subscription.
2. You will answer questions:
- What statuses operation object can get and when they are triggered?
- Does Azure Marketplace store history of changes?

## Additional resources

- https://docs.microsoft.com/en-us/azure/marketplace/partner-center-portal/pc-saas-fulfillment-webhook
- https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-custom-api-authentication
