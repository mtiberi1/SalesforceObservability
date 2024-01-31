# Salesforce Change Data Capture Event Integration with AWS & Grafana

## Course #  

This course provides an in-depth configuration overview to integrate change data capture events from Salesforce to Amazon EventBridge with a utilization of Grafana to visualize these real-time events. 

In this implementation, Salesforce Change Data Capture is being used to publish Salesforce change events. The Salesforce Event Relay service subscribes to the published Salesforce events and relays these events to Amazon EventBridge.


## Prerequisites

### 1. AWS Setup   

  > **Retrieve AWS Account ID & Access Keys:**
  > 1. Under your AWS account name copy the **Account ID**.   
  > 2. Click **Security Credentials** and create **New Access Keys**.   
  > 3. **Save** these credentials for next steps.   

![AWS Account ID](AWSAccountID.png)                            ![AWS Access Keys](AWSAccessKeys.png)


>

### 2. Create a Named Credential # 
A named credential stores your AWS account information. The credential will be used later to set up an event relay. It is created within the Setup Salesforce user interface following the steps below.

> **&#9432;** Salesforce User Permissions Needed: **Customize Application**<br/>                            *[Salesforce Developer Guide: Create a Named Credential](https://help.salesforce.com/s/articleView?id=sf.ev_relay_create_named_credential.htm&type=5)*<br/> 


> 1. From Setup, in the Quick Find box, enter **Named Credentials**, and then select **Named Credentials**.
> 2. Expand the dropdown next to New, and then click **New Legacy**.
> 3. Complete the fields.  
>    - For **Label**, enter AWS US-X_X  
>    - For **Name**, enter AWS_US_X_X  
>    - For **URL**, enter a URL in the format arn:aws:aws_region:aws_account_number.          
>       - Replace the **aws_region** placeholder with your AWS region.      
>       - Replace the **aws_account_number** placeholder with your 12-digit AWS account ID.       
>       - The URL is case-sensitive, and aws_region is in capital letters.  
>       - For example, the URL for an account in the US-WEST-2 region has this format:   
>        *arn:aws:US-WEST-2:XXXXXXXXXXXX. (XXXXXXXXXXXX is a placeholder for the 12-digit AWS account ID.)*  
>    - For an authenticated connection, create an IAM user (or group) with AmazonEventBridgeFullAccess permissions, generate an **access key** and **secret access key** for the user  
>    - Keep **Generate Authorization Header** selected.  
> 4. Save you changes.

![NamedCredential](NamedCredential.png)

>

### 3. Review General Required Permissions for Change Events #
> **&#9432;** Salesforce User Permissions Needed:
[Salesforce Developer Guide: CDC Required Permissions](https://developer.salesforce.com/docs/atlas.en-us.change_data_capture.meta/change_data_capture/cdc_security_perms.htm)
![CDCRequiredPermissions](CDCRequiredPermissions.png)

>

### 4. Review field level and object security for objects being used in Change Data Capture *(by subscribed user's profile)*
 > **&#9432;** [Salesforce Developer Guide: CDC Field-Level Security](https://developer.salesforce.com/docs/atlas.en-us.change_data_capture.meta/change_data_capture/cdc_security_field.htm)

 Change Data Capture(CDC) respects your org’s field-level and object security settings. Delivered events contain only the fields that a subscribed user (user that is used to connect Salesforce to AWS) is allowed to view. Before delivering a change event for an object, ensure the subscribed user’s field and object permissions are checked. If a subscribed user has no access to a field, the field isn’t included in the change event message that the subscriber receives.

>1. From Setup, in the Quick Find box, search for and then select **Profiles**.
>2. Select the profile of the CDC subscribed user *(profile of the user that is used to connect Salesforce to AWS)*.
>3. Select the object you are using for CDC data sync.
>4. Review object and level settings:
>   - Object Permissions: **View All** enabled 
>    - Field Level Permissions: **Read** Access checked for all fields you'd like to see in delivered events
>5. Save the changes to the profile object settings
>6. Repeat for all objects being used in CDC 

![CDCFieldLevelSecurity](CDCFieldLevelSecurity.png)

>

### 5. Set Up and Connect Postman to your Salesforce Org 

>**&#9432;** Fork the [Postman Salesforce Developers APIs Collection](https://www.postman.com/salesforce-developers/workspace/salesforce-developers/folder/12721794-ff8abefd-8a53-444e-bac4-266b5d2c8a72?ctx=documentation)- a resource that wraps together 200+ request templates for many Salesforce Platform APIs. This open source collection uses the configurable Postman environment and variables so that you can easily authenticate and try the requests on multiple Salesforce orgs.

>1.  Open the public [Salesforce Developers](https://www.postman.com/salesforce-developers/workspace/salesforce-developers) workspace to create a fork of the Salesforce APIs collection.
>2.  In Collections, select **Salesforce Platform APIs** to expand it.
>3.  Click **Create Fork** to create a fork of the collection.
>4.  Enter **MySalesforceAPIFork**.
>5.  Keep **SalesforceCollection** as the Workspace.
>6.  Click **Fork Collection**.
![PostmanAPICollection](image.png)

>

### 6. Authorize Your Salesforce Org 
You need to authenticate with Salesforce to access the APIs. Authentication grants you an access token that's valid for a certain duration. Repeat this process whenever your access token expires.

>1.  In Postman, under Collections, **Salesforce Platform APIs** should be selected.
>2.  Be sure No Environment is selected.
>3.  The Authorization tab should be open.
>4.  Type should be **OAuth 2.0**.
![PostmanAuthorizeOrg1](PostmanAuthorizeOrg1.png)

>5. Click **Get New Access Token**. 
![PostmanAuthorizeOrg2](PostmanAuthorizeOrg2.png)

>6. Click **Allow** to let the “Salesforce Platform APIs Collection for Postman” access your Org.
![PostmanAuthorizeOrg3](PostmanAuthorizeOrg3.png)

>7. A success message appears briefly, and then you're redirected to the Manage Access Tokens dialog.
![PostmanAuthorizeOrg4](PostmanAuthorizeOrg4.png)

>8. Verify that the **instance_url** points to your Org. 
>9. Copy the **instance_url** to use in step 12. Be sure to copy only the URL with no extra characters.
>10. Click **Use Token**.
>11. Open **Variables** Tab.
>12. In the**_endpoint** row Current Value column, paste the instance_url value that you copied in step 9. 
>13. Ensure your version variable matches your Org's API version.  [Salesforce Developer Guide: Find Salesforce API Version](https://help.salesforce.com/s/articleView?id=000386929&type=1) 
>14. Click **Save**.

>

### 7. Test Your Connection 
>1. In Collections, select **Salesforce Platform APIs** to expand it.
>2.  Select **REST** to expand it.
>3.  Select **GET Limits**. Limits open in the main panel.
>4.  Click **Send**.
>5.  Verify the status is **200 OK**.

**&#9432;** You may need to turn Follow Authorization header configuration ON to retain authorization header when a redirect happens to a different hostname.

![PostmanConnection](PostmanConnection.png)

>
>
## Create a Salesforce Event Relay Configuration for a Custom Object CDC

### 1. Enable Change Data Capture for the Object
>   1. From  Setup, in the Quick Find box, search and then select **Change Data Capture**.
>   2. Select the object from the **Available Entities** and scroll right, then click Save. The selected object should now appear under the **Selected Entities** list. 

![EnableCDC](EnableCDC.png)


### 2. Create a Channel and Channel Members  

>**&#9432;** **Salesforce User Permissions Needed: Customize Application**   
[Salesforce Developer Guide: Create a Channel & Channel Members](https://help.salesforce.com/s/articleView?id=sf.ev_relay_create_channel_section.htm&type=5) <br/>
[Postman Salesforce Developers APIs Collection](https://www.postman.com/salesforce-developers/workspace/salesforce-developers/folder/12721794-ff8abefd-8a53-444e-bac4-266b5d2c8a72?ctx=documentation) <br/>

>An event relay uses an event channel. Each event relay is associated with a unique channel that’s identified by its channel ID.A channel is a stream of events in the Salesforce event bus.  It contains members that specify the type of events that can be received on the channel. A single channel can contain multiple types of platform events or change data capture events, but not both.  Don’t reuse a channel for another event relay. For a channel of change data capture events, you can select the standard ChangeEvents channel instead of creating a custom channel.  
>   - **Standard Channels:**
>       - The pre-defined standard channel ChangeEvents includes all change events that correspond to the Salesforce objects that you select in Setup. You can select ChangeEvents from the channels dropdown in the event relay creation wizard. Before you can use the standard ChangeEvents channel, from Setup, select the objects that you want change events for in the Change Data Capture page
>  - **Custom Channels:**
>       - You can create a custom channel to receive change event messages. If you create a channel for changes in one Salesforce object, such as AccountChangeEvent, the channel contains one member for AccountChangeEvent. To receive events for another Salesforce object, such as LeadChangeEvent, add another channel member.
 

### 2A. Create a Custom Channel that Holds a Stream of CDC Events
> **&#9432;** Use the correct API version for your SF Org. [Salesforce Developer Guide: Find Salesforce API Version](https://help.salesforce.com/s/articleView?id=000386929&type=1)      

>1. Send a **POST** request to this URI 
>    - /services/data/v59.0/tooling/sobjects/PlatformEventChannel  
>2. If you're using Postman, expand **Event Platform | Custom Channels | Platform Event**, and click **Create channel**.
>3. To configure a channel that receives change event messages, set channelType to data. The channel label appears in the event relays UI. Use this example request body. In Postman, click Body, and replace the body with a formatted JSON body similar to below.     
 
```json   
   {
      "FullName": "RelayChangeEvent_Channel__chn",
		  "Metadata": {   
          "channelType": "data",   
          "label": "Custom Channel for Relaying Change Events"   
      }
    }
   ```
        
>4.  Send the request. The response received looks similar to this response.    

```json    
   {
		  "id": "0YLRM00000001fR4AQ",
		  "success": true,
		  "errors": [],
		  "warnings": [],
		  "infos": []
		}
   ```
		

> 	A channel (PlatformEventChannel) can have multiple channel members (PlatformEventChannelMember), which means that you can add multiple change events to a channel. This example adds only one change event. To add another event to the channel, create another PlatformEventChannelMember.

![CreateChannel](CreateChannel.png)

>
>
### 2B. 	Create a Channel Member to Associate the CDC Event
>1.  Send a **POST** request to this URI
      - /services/data/v59.0/tooling/sobjects/PlatformEventChannelMember
      - Use this example request body:    

   ```json   
   {
			  "FullName": "MyRelayChangeEvent_Channel__chn_ChangeEvent",
			  "Metadata": {
			     "eventChannel": "MyRelayChangeEvent_Channel_chn", 
			     "selectedEntity": "MyRelayChangeEvent__ChangeEvent"
        }
    }
   ```

>**&#9432;**  The format for selected entity for standard objects is <'Object'>ChangeEvent. For custom objects, the format is <'Object'>__ChangeEvent".   


>2.  Send the request. The response received will look similar to this response.       
   ```json    
   {
		  "id": "0v8RM0000000G3iYAE",
		  "success": true,
		  "errors": [],
		  "warnings": [],
		  "infos": []
		}
   ```
  

### 3. Create an Event Relay  

>**&#9432;** **Salesforce User Permissions Needed: Customize Application** <br/> 
[Saleforce Developer Guide: Create an Event Relay](https://help.salesforce.com/s/articleView?id=sf.ev_relay_create_ui.htm&type=5)    

>1. From Setup, in the Quick Find box, search and then select **Event Relays**.
>2. In the Event Relays page, click **New Event Relay**. Use the event relay creation wizard and provide the requested information in each step.
>3. Enter a **label** for the event relay. The label appear in the event relays list view and the event relay detail page. Make sure you choose a meaningful name and try to make it unique. The name is auto populated based on the label that you provide.
>4. Select a **named credential** that contains your AWS account information. 
>5. Select a **channel** that references the events that you want to send to EventBridge. 
>6. Select an **error recovery option**. This option is used when the system can resume from the last relayed event after it recovers from an error. 
>7. Review the summary screen and save your changes to create the event relay.    

  >**&#9432;** After the event relay is created, the event relay detail page contains the Partner Event Source Name field. This field is populated after a short delay (refresh page) with the name of the partner event source that's created in the AmazonBridge.    

![EventRelay](EventRelay.png)   



### 4. Configure AWS    
After configuring the event relay in Salesforce, the Event Relay creates a partner event source in the Amazon EventBridge. The partner event source needs to be associated with an event bus. 

  ### 4A. 	Activate the Partner Event Source in Amazon EventBridge
>1. Login to AWS console.
>2. In the search box, enter **Amazon EventBridge** and then from Services, select **Amazon EventBridge**.
>3. In Amazon Eventbridge, under Integration, select **Partner event sources**. 
>4. In the search box, enter the name of your event source, the **Partner Event Source Name field value** that you copied earlier.
>5. Select your event source, and click **Associate with event bus**.
>6. Click **Associate**. The status of the event source changes to Active.    


  ### 4B. 	Create an EventBridge Rule for Logging Events in the CloudWatch Log 
>1. In Amazon Eventbridge, click **Rules**.
>2. Select your event bus from the dropdown. The name of the event bus is the same as the name of the partner event source that you queried earlier. 
>	   - It’s in this format: aws.partner/salesforce.com/orgID/channelID
>3. In the Rules section, click **Create rule**.
>4. Provide a name for your rule. Click next. 
>5. Under Event source, select **AWS events or EventBridge partner events**.
>6. Skip the Sample event section. 
>7. In Event pattern, for Event source, select **EventBridge partners**.
>8. From the partner dropdown, select **Salesforce**.
>9. For Event type, select **All Events**. The event pattern box autopopulates to this value.   

```json   
    			{
			  "source": [{
			    "prefix": "aws.partner/salesforce.com"
			  }]
			}
```    
>9. The rule matches incoming events based on the defined pattern. If you want to add a filter to match a specific platform event, match events whose detail-type field contains the API name of the platform event. 
>      - Under Event pattern, click **Custom patterns** (JSON editor).
>      - In the input box, In Define pattern, select **Custom pattern**, and replace the pattern with this pattern. Click next.    
					
```json    
    					{
					  "source": [{
					    "prefix": "aws.partner/salesforce.com"
					  }],
					  "detail-type": ["SObject_APIName__e"]
					}
```   
>10. Under Target 1, Target types, select **AWS service**.
>11. Under Select a target, select **CloudWatch log group**.
>12. Complete the log group path. *For example: /aws/events/mygroup/log.*
>13. Click Next and then Next.
>14. Review the rule that you created, and then click **Create rule**.


  ### 4C. Verify Receiving Events in CloudWatch 
>1. Click the rule that you just created.
>2. Under Targets, click the log. The CloudWatch log opens in a new tab.
>3. Publish an event from Salesforce. *In this example, we publish an event message with REST API.*
>      - Perform a POST request to this URI. If using Postman, click **REST > SObject > SObject Create**
>           - /services/data/v58.0/sobjects/SObject_APIName__e/
>4. You should receive a success response. The status of OPERATION_ENQUEUED means that the platform event message is published asynchronously.
>5. Refresh the CloudWatch log stream. The received event is displayed in the log group 


  ### 5. Activate the Event Relay Confirguration in Salesforce
>1. Start the Event Relay either from the event relays page fo the event relay detail page. 
>      - In the Event Relays page, click the dropdown under the Actions column, and select **Edit**. 
>      - In the Event Relay detail page, click **Edit**. 
>2. In the Edit window, in the State dopdown, select **Run**. 
>3. Save your changes. 

![ActivateEventRelay](ActivateEventRelay.png)


  ### 6. Test






	











