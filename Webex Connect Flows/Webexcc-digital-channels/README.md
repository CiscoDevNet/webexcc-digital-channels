# Webex Contact Center Digital Channel Flows

This provides template workflows and sample workflows for Webex Contact Center and Webex Connect integration. All workflows are versioned and Webex Contact Center deployment **region agnostic**.


## Overview

Digital flows are configured in Webex Connect to drive the full lifecycle of a digital contact — from the moment a customer initiates a conversation to the point the interaction is closed. Each flow is composed of a series of **nodes**, where each node performs a specific function along the contact handling path. Nodes are connected to define both the success path (what happens when everything works as expected) and the error path (what happens when something fails).

For Contact Center to operate correctly, certain nodes must appear in a specific order. The templates and samples in this repository illustrate the correct arrangement and configuration of those nodes.

## Template Flow Node Reference

Template flows are divided into two categories: **Media Specific Workflows** and **Event Handling Workflows**. The following sections describe what each node in these template flows does.


### Media Specific Workflows

These flows are triggered by inbound customer activity on a specific channel (Live Chat, Email, SMS, Facebook, WhatsApp, or Apple Messages for Business). One flow instance must be imported and configured per channel asset.

#### 1. Start Node

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
