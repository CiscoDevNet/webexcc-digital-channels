# webexcc-digital-channels

This repo provides the template workflows and sample workflows for Webex Contact Center and IMI integration. These workflows 
are versioned and Webex Contact Center deployment **region agnostic**. 

## Digital Flows
- Digital flows are configured in Webex Connect to drive the lifecycle of a digital contact.
- Digital flows contain various nodes that enable you to configure success and error paths for the execution of customer interactions.
- For Contact Center to function, certain nodes need to be arranged in specific order, hence templates and samples are provided to depict the correct usage of the flows.

## 3.1 Workflows
In 3.1 version of workflows, the flows are enhanced with Security scan results that are passed on to the backend to further protect your contact centre against malware content coming in via attachments in messages.

### Updates available in 3.1 Workflows
- Support for:
  - Utilising PCI, Malware and Security scan results
  - Receiving emails without sender name / subject
  - Plain text emails

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

### Template Flows:-

#### Media Specific Flows:-
- This folder contains media specific template flows like Email Inbound Flow, Facebook Inbound Flow, Live Chat Close Flow, Live Chat Inbound Flow, Live Chat Inbound Flow Without Form, SMS Inbound Flow and WhatsApp Inbound Flow.

#### Event Handling Flows:-
- This folder contains event handling template flows. It contains Task Close Flow, Task Modified Flow, Task Routed Flow.

### Steps to perform Manual Upgrade from v3.0 to v3.1
- As described in file `Webex Connect Flows/How to manually upgrade from v3.0 to v3.1 workflows.MD`

## Version History

| Workflows version | High level Description                      |
| ----------------- | ------------------------------------------- |
| 3.1               | Simplified flows with Security scan results |
| 3.0               | Simplified flows                            |