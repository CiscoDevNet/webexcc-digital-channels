# webexcc-digital-channels

This repo provides the template workflows for Webex Contact Center and IMI integration. 

## Workflows Overview
The following workflow templates are included : 

Any new nodes need to be added to the flow needing taskId or conversationId as an input can use the following pre-defined custom variables $(taskId) or $(conversationId) instead of $(node_id.taskId) or $(node_id.conversationId)

1. Media Specific Workflows : These flows need to be imported once per asset of the specific channel.
    * Facebook Inbound Message Workflow - This workflow will be triggered for every inbound customer message over the integrated Facebook page. 
    * Live Chat Inbound Message Workflow - This workflow will be triggered for every inbound customer message over the configured customer chat widget.
    * SMS Inbound Message Workflow - This workflow will be triggered for every inbound customer sms sent over the integrated SMS phone number.
    * Email Inbound Message Workflow - This workflow will be triggered for every inbound customer email to the integrated email account.
    * Whatsapp Inbound Message Workflow - This workflow will be triggered for every inbound customer message over the integrated whatsapp business number.
    

2. Media Agnostic Workflows : These flows need to be imported only * once per Organization and is applicable for all channels. If complex shared flows are already available in
   the same org then please refer to the migration strategy section.
   These flows are optional after simplification. They can be configured if flow admin needs to build custom logic like displaying a Screen pop on the Agent's desktop etc. on occurrence of certain events.
    * Task Routed Workflow - This workflow will be triggered when an agent is assigned to a Task (typically after a Queue Task request and agent accepting the task).
    * Task Modified Workflow - This workflow will be triggered when an agent initiates a conversation transfer request or a conference request.
    * Close Task Workflow -  This workflow will be triggered when an agent or system ends a Task.
    
## Quick start on Workflows

* Follow the process for Organization setup in WxCC with IMI Integration.
* Download the sample flow template from the repository.
* Import the flows in your Webex Connect service.
* Add authorizations at integration level.
* For Resolve Conversation node flow id needs to be updated manually. Flow id can be obtained from the address bar. For example flow id (41896) can be obtained from the url https://testorg.datacenter.webexconnect.io/flowdesigner/flow/v3/flowView?flowId=41896 
* Modify the flows as per your use case.
* Make flows live with the configured assets.

Note : Queue selection from queue task node is mandatory for any channel flows before making flow live.

## Migration Strategy
* Flow admins can manually migrate the existing main flows for each asset to the simplified version at their pace.
* The shared flows can be manually migrated to the simplified versions only when all the main flows for each asset are migrated already.
* If the org has migrated only a few of their main flows to simplified version, shared complex flow should have the condition to prevent running for contacts created via simplified flows and similarly simplified shared flows should have the condition to run only for the assets migrated to simplified versions.
* For migration please refer to the steps mentioned here https://github.com/CiscoDevNet/webexcc-digital-channels/blob/main/Webex%20Connect%20Simplified%20Flows/IMI%20Flow%20Migration%20Strategy%20Guide.pdf


Details of this material is available in Webex Connect platform documentation.
* Login to your Webex Connect account
* Click on `Documentation` under `Help` section
* Webex Connect platform documentation will open in a new tab
* Under `GETTING STARTED` click on `Webex Connect Platform Overview`
* Scroll down to `CISCO WEBEX CONTACT CENTER AND Webex CONNECT INTEGRATION` section
* Click on `Overview`
