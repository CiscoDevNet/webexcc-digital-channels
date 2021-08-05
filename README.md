# webexcc-digital-channels

This repo provides the templates workflows for Webex Contact Center and IMI integration. These workflows 
are versioned and specific to Webex Contact Center deployment regions.

## Workflows Overview
Inside region specific folders, the following workflow templates are included : 

1. Media Specific Workflows : (Need to be updated once per asset of the specific channel) 
    * FBM Inbound Message Workflow - This workflow will be triggered for every inbound customer message over the integrated Facebook page. 
    * Live Chat Inbound Message Workflow - This workflow will be triggered for every inbound customer message over the configured customer chat widget.
    * SMS Inbound Message Workflow - This workflow will be triggered for every inbound customer sms sent over the integrated SMS phone number.
    * Email Inbound Workflow - This workflow will be triggered for every inbound customer email to the integrated email account.    
    
2. Media Agnostic Workflows : (Need to be impoted only once per Organization and is applicable for all channels)
    * Task Routed Workflow - This workflow will be triggered when an agent is assigned to a Task (typically after a Queue Task request and agent accepting the task).
    * Task Modified Workflow - This workflow will be triggered when an agent initiates a conversation transfer request or a conference request.
    * Close Task Workflow -  This workflow will be triggered when an agent or system ends a Task.

## [Quick start on Workflows](https://help.imiconnect.io/docs/wxcc-flow-configuration-using-sample-templates)

* `Follow the process for Organization setup in WxCC with IMI Integration.`
* `Download the flows for the respective deployment data center.`
* `Import the flows in your IMI connect service.`
* `Add authorizations for the flow nodes.`
* `Modify the flows as per your use case.`
* `Make flows live with the configured assets.`

Details of this material is available at: [CISCO WEBEX CONTACT CENTER AND IMICONNECT INTEGRATION](https://help.imiconnect.io/docs/wxcc-overview)
