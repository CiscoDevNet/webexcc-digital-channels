# webexcc-digital-channels

This repo provides the template workflows for Webex Contact Center and IMI integration.

## Workflows Overview
The following workflow templates are included :

1. **Media Specific Workflows** : These flows need to be imported once per asset of the specific channel.
    * **Facebook Inbound Flow** - This workflow will be triggered for every inbound customer message over the integrated Facebook page.
    * **Live Chat Inbound Flow** - This workflow will be triggered for every inbound customer message over the configured customer chat widget, and the widget has a live chat form configured.
    * **Live Chat Inbound Flow Without Form** - This workflow will be triggered for every inbound customer message over the configured customer chat widget and the widget doesn't have a live chat form configured.
    * **SMS Inbound Flow** - This workflow will be triggered for every inbound customer sms sent over the integrated SMS phone number.
    * **Email Inbound Flow** - This workflow will be triggered for every inbound customer email to the integrated email account.
    * **Whatsapp Inbound Flow** - This workflow will be triggered for every inbound customer message over the integrated whatsapp business number.
    * **Live Chat Close Flow** - This workflow will be triggered when a livechat customer has ended the chat.


2. **Event Handling Workflows** : These flows are optional unless flow admin needs to build custom logic like displaying a Screen pop on the Agent's desktop etc. on occurrence of certain events.With v3.0 flows there will be default handling for all the events like
    adding participant , removing participant , closing the conversation etc. If flow admin needs any alternative handling or additional business logic then these flows can be imported only once per organization and customized as per the requirement.
    * **Task Routed Flow** - This workflow will be triggered when an agent is assigned to a Task (typically after a Queue Task request and agent has accepted the task).
    * **Task Modified Flow** - This workflow will be triggered when an agent has initiated a conversation transfer request or a conference request.
    * **Task Closed Flow** -  This workflow will be triggered when an agent or system has ended a Task.
   
    These flows are applicable for all channels by default. If v2.x shared flows are already available in the same org, refer to the migration strategy section.

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
* Flow admins can manually migrate the existing main flows for each asset to V3.0 at their pace.
* The shared flows can be manually migrated to the V3.0 versions only when all the main flows for each asset are migrated already.
* If the org has migrated only a few of their main flows to V3.0, shared V2.x flows should have the condition to prevent running for contacts created via V3.0 flows. Similarly, V3.0 shared flows should have the condition to run only for the assets migrated to V3.0.
* For migration, refer to the steps mentioned in 
  [Migration Strategy](Migration Strategy.pdf)

Details of this material is available in Webex Connect platform documentation.
* Login to your Webex Connect account.
* Click `Documentation` under `Help` section.
* Click on `Documentation` under `Help` section.
* Webex Connect platform documentation will open in a new tab.
* Webex Connect platform documentation will open in a new tab.
* Under `Getting Started`, click `Webex Connect Platform Overview`.
* Under `GETTING STARTED` click on `Webex Connect Platform Overview`.
* Scroll down to the `Webex Contact Center and Webex Connect Integration` section.
* Scroll down to `CISCO WEBEX CONTACT CENTER AND Webex CONNECT INTEGRATION` section.
* Click `Overview`.
