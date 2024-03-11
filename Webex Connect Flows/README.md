# webexcc-digital-channels

This repo provides the template workflows and sample workflows for Webex Contact Center and IMI integration. These workflows 
are versioned and Webex Contact Center deployment **region agnostic**. 

## Digital Flows
- Digital flows are configured in Webex Connect to drive the lifecycle of a digital contact.
- Digital flows contain various nodes that enable you to configure success and error paths for the execution of customer interactions.
- For Contact Center to function, certain nodes need to be arranged in specific order, hence templates and samples are provided to depict the correct usage of the flows.

## 3.2 Workflows
In 3.2 version of workflows, enhancements to the flows allow for setting contact priority.

### Updates available in 3.2 Workflows
- Support for:
  - Queue the contact based on priority.
  - Receiving emails from flows with the asset email as the sender's address.

### Sample Flows:-

#### Attachment Drop notification to customers
- It demonstrates how the flow admin can configure the flows to notify the customer if and when an attachment they sent is dropped and why.

#### Bot Flows:-
- It contains three sample bot flows.
- Facebook Task Bot Inbound Flow depicts the usage of a Task Bot.
- LiveChat and SMS Q&A bot Inbound Flow depicts the usage of a Q&A Bot.

#### Usage of PIQ And EWT In Flows:-
- It demonstrates the usage of PIQ and EWT via Live Chat Inbound Sample Flow.

#### Usage of Screen Pop in Flows:-
- It demonstrates the usage of set variable via Task Close Sample Flow.

#### Usage of Set Variable in Flows:-
- It demonstrates the usage of set variable via Live Chat Sample Flow.

#### Usage of Contact Priority in Flows:-
- It demonstrates the usage of contact priority via Email Flow.

### Template Flows:-

#### Media Specific Flows:-
- This folder contains media specific template flows like Email Inbound Flow, Facebook Inbound Flow, Live Chat Close Flow, Live Chat Inbound Flow, Live Chat Inbound Flow Without Form, SMS Inbound Flow and WhatsApp Inbound Flow.

#### Event Handling Flows:-
- This folder contains event handling template flows. It contains Task Close Flow, Task Modified Flow, Task Routed Flow.

### Steps to perform Manual Upgrade from v3.1 to v3.2
- As described in file `Webex Connect Flows/How to manually upgrade from v3.1 to v3.2 workflows.MD`

## Version History

| Workflows version | High level Description                                                                                                                                                                                                                                  |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 3.2               | Flow nodes have been updated to enable setting contact priority.                                                                                                                                                                                        |
| 3.1               | The malware scanning feature has been integrated into all attachments sent across various channels. The channel start nodes have been improved to include a new set of variables for identifying malware and presenting security scan results.          |
| 3.0               | The 2.x flows undergone a major update in 3.0 where the contact center specific logic that used to be a part of flow got replaced with the Resolve Conversation node. Shared flows like Task Routed, Task Modified and Task Closed are also simplified. |