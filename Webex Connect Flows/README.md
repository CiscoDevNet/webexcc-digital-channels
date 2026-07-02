# Webex Contact Center Digital Channel Flows

This repository provides template workflows and sample workflows for Webex Contact Center and Webex Connect integration. All workflows are versioned and Webex Contact Center deployment **region agnostic**.

---

## Overview

Digital flows are configured in Webex Connect to drive the full lifecycle of a digital contact — from the moment a customer initiates a conversation to the point the interaction is closed. Each flow is composed of a series of **nodes**, where each node performs a specific function along the contact handling path. Nodes are connected to define both the success path (what happens when everything works as expected) and the error path (what happens when something fails).

For Contact Center to operate correctly, certain nodes must appear in a specific order. The templates and samples in this repository illustrate the correct arrangement and configuration of those nodes.

---

## Template Flow Node Reference

Template flows are divided into two categories: **Media Specific Workflows** and **Event Handling Workflows**. The following sections describe what each node in these template flows does.

---

### Media Specific Workflows

These flows are triggered by inbound customer activity on a specific channel (Live Chat, Email, SMS, Facebook, WhatsApp, or Apple Messages for Business). One flow instance must be imported and configured per channel asset.

#### 1. Channel Start Node

The Channel Start Node is the entry point for every inbound flow. It fires when a new customer message or session arrives on the channel. Depending on the channel, the node is named differently (e.g., _Live Chat/In-App Messaging Node_, _Facebook Message Node_, _SMS Inbound Node_, _Email Inbound Node_, _WhatsApp Node_). The node outputs a set of variables — such as the customer's identifier, message content, attachment metadata, and malware scan results — that downstream nodes can reference. From v3.1 onward, the start node also exposes variables indicating whether any attachment contains malware so the flow can branch accordingly.

To restrict a flow to a specific asset or channel, conditions can be enabled on the start node using fields such as `webex.mediaChannel` (e.g., `"web"` for Live Chat) or `webex.destination` (the App ID or service number of the asset).

#### 2. Set Variable Node

The Set Variable Node stores key pieces of information into named flow variables so they can be referenced later in the flow. Typical variables set here include `customerName`, `customerEmail`, `appId`, `liveChatDomain`, `FBpageid`, `WANumber`, and `bizemailid`. These variables must be populated with the correct asset-specific values before the flow is made live.

When processing a Live Chat form submission, this node is also used to extract form field values from the channel start node's output — for example, `$(n2438.inappmessaging.formFields.Name)` — and map them to the standardised `customerName` and `customerEmail` variables that appear on the Agent Desktop.

#### 3. Receive Node

The Receive Node pauses the flow and waits for a response from the customer or the channel (such as a form submission). After the customer submits their pre-chat form, the Receive Node captures the response. **Transition Actions** on the Receive Node are used to extract form field values and assign them to flow variables like `customerName` and `customerEmail`. These values are then visible to the agent on the Agent Desktop. The specific variable expressions used depend on the names of the fields configured in the Live Chat template (e.g., `$(n2438.inappmessaging.formFields.UserName)`).

#### 4. Evaluate Node

The Evaluate Node executes JavaScript expressions to derive or transform values within the flow. In Media Specific Workflows it is commonly used to parse attachment metadata — for example, to determine whether a file's MIME type is `message/rfc822` and should be excluded from processing, or to extract inline image `cid` parameters to forward them to the Engage API. In Event Handling Workflows, the Evaluate Node extracts all predefined system variables from the incoming request JSON and assigns them to usable flow variables. A helper method is also provided within the Evaluate Node to retrieve flow-scoped and global variables that were set in the main inbound flows via the Set Variable Node.

#### 5. Branch Node

The Branch Node introduces conditional logic into the flow. It evaluates one or more conditions against flow variable values and routes execution down a matching branch. For example, in the Task Modified Flow the Branch Node checks the value of `webex.context` to determine whether a participant was added or removed, and then executes the appropriate screen pop or HTTP call. In Media Specific Workflows it can be used to check malware scan results and route safe attachments down one path while rejecting unsafe ones on another.

#### 6. Queue Task Node

The Queue Task Node is mandatory in every inbound channel flow. It submits the incoming contact to the Webex Contact Center routing engine and places it in a queue for agent assignment. Before making a flow live, the flow administrator must configure this node with the correct queue selection. Failure to configure the queue will prevent the flow from routing contacts to agents.

#### 7. Resolve Conversation Node

The Resolve Conversation Node handles the Contact Center–specific logic required to close a conversation record in the backend. It acts as the unified termination point that replaces the more complex multi-step logic found in v2.x flows. The node requires the **Flow ID** of the current flow to be set manually — the Flow ID can be obtained from the address bar of the Flow Designer (e.g., `flowId=41896` in the URL `https://testorg.datacenter.webexconnect.io/flowdesigner/flow/v3/flowView?flowId=41896`). Note that `message/rfc822` MIME type attachments should not be passed to this node, as documented in the platform limitation addressed in v3.5.

#### 8. Close Task Node

The Close Task Node terminates the associated task in the Webex Contact Center backend. It is used primarily in the **Live Chat Close Flow**, which is triggered when a customer clicks the "End chat" button in the chat widget. The node requires a task identifier, which is typically the `aliasId` value retrieved from the Search Conversation response earlier in the flow.

#### 9. Search Conversation Node

The Search Conversation Node queries the Webex Connect conversation store to locate an existing conversation record using a known identifier (such as the customer's session key or channel-specific identifier). Its primary output — the `aliasId` — is passed downstream to the Close Task Node to ensure the correct task is terminated when a customer ends a Live Chat session.

---

### Event Handling Workflows

These flows are **optional** and are triggered by Webex Contact Center task lifecycle events rather than by customer-initiated messages. They are media-channel agnostic, meaning a single deployed instance covers all channels unless conditions are added to restrict it. If custom logic such as Screen Pop, HTTP API calls, or CRM updates is needed on a specific event, these flows can be imported, customised, and deployed in multiples — one per channel or asset.

#### 1. WxCC Task V2 Start Node

The WxCC Task V2 Start Node is the entry point for all Event Handling Workflows. The event type is configured in this node and determines which platform event triggers the flow:

- **Task Routed** — fires when an agent is successfully assigned to a contact and accepts it. Used in the Task Routed Flow.
- **Task Modified** — fires when a conversation transfer or conference is performed. Used in the Task Modified Flow.
- **Task Closed** — fires when an agent or the system ends a contact and the conversation is closed. Used in the Task Close Flow.

Output variables exposed by this node (such as `webex.variables`, `webex.RequestBody`, `webex.context`, `webex.mediaChannel`, and `webex.destination`) are available for use in downstream nodes. Because `webex.variables` and `webex.RequestBody` are nested variables, the Transition Actions on this node assign them to flat variables (`response` and `requestBody` respectively) for easier reference throughout the rest of the flow.

To scope an event flow to a specific channel or asset, the conditions checkbox on this node can be enabled. Conditions can filter by `webex.mediaChannel` (e.g., `"web"` for Live Chat) or by `webex.destination` (the App ID or service number). Multiple assets can be targeted using OR-combined `equals` conditions; assets can be excluded using AND-combined `notequals` conditions. There is no limit to the number of event flows that can be deployed for the same event type.

#### 2. Evaluate Node (Event Flows)

In Event Handling Workflows, the Evaluate Node immediately follows the WxCC Task V2 Start Node. Its role is to parse the incoming request body and extract all predefined system variables into usable flow-scoped variables. It also includes a helper method that retrieves any custom flow variables or global variables that were set in the main inbound flows using the Set Variable Node, making those values available for use in screen pops, HTTP calls, or other business logic.

#### 3. Branch Node (Event Flows)

The Branch Node in event flows routes execution based on the value of extracted variables. In the Task Modified Flow, for example, it evaluates `webex.context` to determine the type of modification — `"add"` (participant added) or `"remove"` (participant removed) — and triggers the appropriate action such as a Screen Pop notification on the Agent Desktop.

#### 4. HTTP Request Node

The HTTP Request Node makes outbound API calls to external systems. It is used in event flows to implement custom logic such as triggering a Screen Pop on the Agent Desktop, updating a CRM record, or calling any third-party REST API when a task lifecycle event occurs. The node is configured with the target URL, HTTP method, request headers, and a payload body that can reference any flow variables extracted by the Evaluate Node.

---

## Version History

---

## v3.5

### What Changed

The v3.5 release focuses on correctness fixes in the Email Inbound Flow to align with platform-documented limitations.

- **Email Inbound Flow — Attachment MIME type filtering:** The Evaluate Node in the Email Inbound Flow was updated to check each attachment's MIME type before passing it to the Resolve Conversation Node. Attachments with a `message/rfc822` type (raw email message attachments) are now filtered out and excluded from the Resolve Conversation Node call, avoiding an error that occurred when such attachments were passed through.

- **Email Inbound Flow — Inline image CID forwarding:** The flow was updated to extract and forward the `cid` (Content-ID) parameter from inline image attachments to the Engage API. This ensures that inline images embedded within email bodies are correctly referenced and rendered in the conversation.

### Steps to Upgrade

Refer to [How to manually upgrade from v3.4 to v3.5 workflows](How%20to%20manually%20upgrade%20from%20v3.4%20to%20v3.5%20workflows.MD)

---

## v3.4

### What Changed

The v3.4 release updates all flows to use the latest available node versions and adds support for proactive chat in the Live Chat Inbound Flow.

- **Node version updates:** All flows were reviewed and updated to reference the latest versions of each node type, ensuring compatibility with current Webex Connect platform capabilities.

- **Live Chat Inbound Flow — Proactive chat support:** A new variant of the Live Chat Inbound Flow was introduced that supports proactive chat messages. This allows the contact center to initiate a chat interaction with a customer rather than waiting for an inbound message.

- **Media Specific Flows:** The email inbound flow received the latest node version updates. The remaining digital channel flows were simplified to reduce complexity while maintaining full functionality.

### Steps to Upgrade

Refer to [How to manually upgrade from v3.3 to v3.4 workflows](How%20to%20manually%20upgrade%20from%20v3.3%20to%20v3.4%20workflows.MD)

---

## v3.3

### What Changed

The v3.3 release extends post-interaction capabilities and adds Apple Messages for Business as a supported channel.

- **Survey delivery:** Flows were enhanced to support sending survey links and survey messages to customers after the agent ends an interaction. The Live Chat Close Flow and Facebook Close Flow were updated to demonstrate this capability via the Usage of Surveys sample flows.

- **Apple Messages for Business:** Full channel support was introduced for Apple Messages for Business. A new Apple Messages for Business Inbound Flow template is included, along with sample flows demonstrating Rich Message features (Time Picker, List Picker) and routing based on entry point parameters (`intentID` and `groupID`).

### Sample Flows Included

- **Attachment Drop Notification to Customer:** Demonstrates how to notify a customer when their attachment is dropped, including the reason, using a branch on the malware scan result or MIME type check.
- **Bot Flows:** Three sample bot flows are included — a Facebook Task Bot Inbound Flow demonstrating Task Bot usage, and a Live Chat / SMS Q&A Bot Inbound Flow demonstrating Q&A Bot usage.
- **Usage of PIQ and EWT:** Demonstrates how to surface Position in Queue (PIQ) and Estimated Wait Time (EWT) values to customers via a Live Chat Inbound Sample Flow.
- **Usage of Screen Pop:** Demonstrates how to trigger a screen pop on the Agent Desktop using a Task Close Sample Flow.
- **Usage of Set Variable:** Demonstrates how to store and reference custom data using a Live Chat Sample Flow.
- **Usage of Contact Priority:** Demonstrates how to assign contact priority during queueing via an Email Flow.
- **Usage of Surveys:** Demonstrates how to deliver a survey after an interaction closes via a Live Chat Close Flow and a Facebook Close Flow.
- **Apple Business Messages Sample Flows:** Demonstrates Rich Message features (Time Picker, List Picker) and entry-point parameter–based routing.

### Steps to Upgrade

Refer to [How to manually upgrade from v3.2 to v3.3 workflows](How%20to%20manually%20upgrade%20from%20v3.2%20to%20v3.3%20workflows.MD)

---

## v3.2

### What Changed

The v3.2 release introduces contact priority support and a minor email sender address improvement.

- **Contact Priority:** The Queue Task Node was updated to accept a contact priority value. Flow administrators can now set the priority of a contact when it is submitted to the queue, allowing higher-priority contacts to be routed to agents before lower-priority ones. The Usage of Contact Priority sample flow demonstrates this via an Email Flow.

- **Email sender address:** Flows were updated to support receiving emails from flows where the sending address matches the asset's configured email address, improving routing accuracy for email-based workflows.

### Steps to Upgrade

Refer to [How to manually upgrade from v3.1 to v3.2 workflows](How%20to%20manually%20upgrade%20from%20v3.1%20to%20v3.2%20workflows.MD)

---

## v3.1

### What Changed

The v3.1 release adds malware scanning for attachments and several channel-specific improvements.

- **Malware scanning — Channel Start Node enhancements:** The Channel Start Nodes for all supported channels were updated to expose a new set of output variables that carry the results of the platform's built-in malware scan. These variables indicate whether an attachment was scanned and what the security verdict was. Flow administrators can use a Branch Node after the Channel Start Node to check these values and route unsafe attachments down an error path rather than processing them further. Only attachments that pass the security scan proceed down the success path.

- **Live Chat / Chat:** The flow was updated so that `customerName` and `customerEmail` values collected via the pre-chat form are correctly passed through the Receive Node Transition Actions and surfaced on the Agent Desktop.

- **Email:** Three edge cases were addressed — emails sent without a subject line, emails sent without a sender name, and plain-text emails (with no HTML body) are now handled without errors.

- **Bot flows:** Minor import errors present in v3.0 bot flows were resolved so flows can be imported cleanly.

### Steps to Upgrade

Refer to [How to manually upgrade from v3.0 to v3.1 workflows](How%20to%20manually%20upgrade%20from%20v3.0%20to%20v3.1%20workflows.MD)

---

## v3.0

### What Changed

The v3.0 release is a foundational simplification of the overall flow architecture.

- **Resolve Conversation Node — Simplified flow logic:** Contact Center–specific backend logic that previously required multiple nodes and manual configuration steps in each flow was consolidated into a single **Resolve Conversation Node**. This node handles the full termination lifecycle of a conversation (closing the task, resolving the conversation record, and performing any required backend callbacks) with minimal configuration. Flow administrators only need to provide the Flow ID to the node.

- **Optional Event Handling Flows:** The shared Task Routed, Task Modified, and Task Closed flows are now fully optional. Webex Contact Center provides default built-in handling for all task lifecycle events. Flow administrators only need to deploy event flows if they require custom business logic (such as Screen Pop or HTTP API calls) on top of the default behaviour.

- **Region-agnostic flows:** All flows were redesigned to be deployment region agnostic, removing hardcoded region-specific endpoints and allowing the same flow to be used across any Webex Contact Center deployment region.
