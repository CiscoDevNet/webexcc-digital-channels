# webexcc-digital-channels

This repo provides the templates workflows for Webex Contact Center and IMI Bot integration. These workflows 
are versioned and Webex Contact Center deployment **region agnostic**. In this new version of flows, the authorization mechanism is centralized and no need to configure it for every node of each flow.

Earlier version of flows which are region specific and required authorization at each node level. 

## Workflows Overview
The following workflow templates are included : 

1. Media Specific Workflows : (Need to be updated once per asset of the specific channel) 
    * FBM Inbound Message Workflow - This workflow will be triggered for every inbound customer message over the integrated Facebook page. 
    * Live Chat Inbound Message Workflow - This workflow will be triggered for every inbound customer message over the configured customer chat widget.
    

## Quick start on Workflows

* Follow the process for Organization setup in WxCC with IMI Integration
* Download the flows template from the Repo
* Import the flows in your IMI connect service
* Add authorizations at integration level
* Modify the flows as per your use case
* Make flows live with the configured assets

Details of this material is available in IMI connect platform documentation.
* Login to your IMI connect account
* Click on `Documentation` under `Help` section
* IMI connect platform documentation will open in a new tab
* Under `GETTING STARTED` click on `imiconnect Platform Overview`
* Scroll down to `CISCO WEBEX CONTACT CENTER AND IMICONNECT INTEGRATION` section
* Click on `Overview`
