# webexcc-digital-channels

This repo provides the template workflows and sample workflows for Webex Contact Center and IMI integration. These workflows 
are versioned and Webex Contact Center deployment **region agnostic**. 

## Digital Flows
- Digital flows are configured in Webex Connect to drive the lifecycle of a digital contact.
- Digital flows contain various nodes that enable you to configure success and error paths for the execution of customer interactions.
- For Contact Center to function, certain nodes need to be arranged in specific order, hence templates and samples are provided to depict the correct usage of the flows.

## 3.3 Workflows
In 3.3 version of workflows, support for Apple Business Messages as a channel has been provided.

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

#### Usage of Survey in Flows:-
- It demonstrates the usage of survey via Live Chat Close Flow and Facebook Close flow.

#### Usage of Contact Priority in Flows:-
- It demonstrates the usage of contact priority via Email Flow.

#### Apple Business Messages Sample Flows
- Contains sample flows to demonstrate usage of
  - Rich Message features supported by Apple such as Time picker, List picker etc. 
  - Routing based on entrypoint parameters i.e. intentID and groupID  

### Template Flows:-

#### Media Specific Flows:-
- This folder contains media specific template flows like Email Inbound Flow, Facebook Inbound Flow, Live Chat Close Flow, Live Chat Inbound Flow, Live Chat Inbound Flow Without Form, SMS Inbound Flow and WhatsApp Inbound Flow.

#### Event Handling Flows:-
- This folder contains event handling template flows. It contains Task Close Flow, Task Modified Flow, Task Routed Flow.

### Steps to perform Manual Upgrade from v3.2 to v3.3
- As described in file `Webex Connect Flows/How to manually upgrade from v3.2 to v3.3 workflows.MD`

