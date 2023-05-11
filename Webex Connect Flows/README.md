# webexcc-digital-channels

This repo provides the templates workflows and sample workflows for Webex Contact Center and IMI integration. These workflows 
are versioned and Webex Contact Center deployment **region agnostic**. 

## Digital Flows
- Digital flows are configured in Webex Connect to drive the lifecycle of a digital contact.
- Digital flows contain various nodes that enable you to configure success and error paths for the execution of customer interactions.
- For Contact Center to function, some nodes need to be arranged in specific order, hence templates and samples are provided to depict the correct usage of the flows.

## 3.0 Workflows
In 3.0 version of workflows we have simplified the flows to ease the lifecycle of a flow admin.
v3.0 folder contains the following flows:-
### Sample Flows:-
#### Bot Flows:-
- It contains three sample bot flows.
- Facebook Task Bot Inbound Flow depicts the usage of a Task Bot. 
- LiveChat and SMS Q&A bot Inbound Flow depicts the usage of a Q&A Bot.

#### Usage of PIQ And EWT In Flows:-
- It demonstrates the usage of PIQ and EWT via Live Chat Inbound Sample Flow.

#### Usage of Screen Pop In Flows:-
- It demonstrates the usage of screen pop via Task Close Flow.

#### Usage of Set Variable in Flows:-
- It demonstrates the usage of set variable via Live Chat Flow and Task Routed Sample Flow.

### Template Flows:-
#### Media Specific Flows:-
- This folder contains media specific template flows. It contains Email Inbound Flow, Facebook Inbound Flow, Live Chat Close Flow, Live Chat Inbound Flow,
  Live Chat Inbound Flow Without Form, Sms Inbound Flow, Whatsapp Inbound Flow.

#### Event Handling Flows:-
- This folder contains event handling template flows. It contains Task Close Flow, Task Modified Flow, Task Routed Flow.
