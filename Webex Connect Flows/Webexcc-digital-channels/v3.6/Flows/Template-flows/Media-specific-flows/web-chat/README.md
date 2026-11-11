# Media Specific Workflows

## Workflows Overview

The following workflow templates are included:

**Media Specific Workflows**: These flows need to be imported once per asset of the specific channel.

- **Web Chat Inbound Flow** - This workflow handles inbound customer conversations received from the web chat widget.
- **Web Chat Proactive Flow** - This workflow handles proactive customer conversations where the chat is initiated by a proactive campaign or rule.
- **Web Chat Close Flow** - This workflow is triggered when a customer ends the web chat session.

Note :

- Queue selection from queue task node is mandatory for any channel flows before making the flow live.
- Flow variables must be updated in the setting section with the correct asset details before making the flow live.

## Updating Flow Variables

Flow variables that need to be updated under Custom Variables for the Web Chat flow:

- Web Chat: `appId` and `webChatDomain`

  ![Web chat Settings](../../../images/Settings.png)

### Web Chat Inbound Flow Details

- In the Web Chat flow, we need to select the template. For creation of template refer to https://help.imiconnect.io/docs/livechat-template
  Sample template is shown in below image

  <br><img width="800" alt="Template" src="../../../images//Template.png"><br>

- In the Web Chat form configuration, we have selected the message type as form and we have to choose the template from the dropdown which has the list of all the web chat templates available in that tenant

  <br><img width="800" alt="formdropdown" src="../../../images//formdropdown.png"><br>

- In this sample we have filled the template as WebChatTemplate (the sample one shown in above image)

  <br><img width="800" alt="Prechatform" src="../../../images/Prechatform.png"><br>

- In the Receive node transition action we need to set the variable customerName and customerEmail and this customerName and customerEmail will
  be visible on the agent desktop.

- We are getting the value for customerName and customerEmail from the web chat form.
- The value of customerName and customerEmail will be extracted from the web chat form response. In the sample web chat template, "Name" and "Email" fields are configured, and these values should be mapped into the standard flow variables for customerName and customerEmail.
  Based on your form fields, the value for the above variables will change, however; the variable names i.e customerName and customerEmail must remain intact.
- For instance, if the form fields are named as UserName and UserEmail, the customerName and customerEmail variables must be mapped from the corresponding form field values in the Receive node Transition Actions.

  <br><img width="800" alt="Receive" src="../../../images/Receive.png"><br>

### Web Chat Close Flow Details

- This flow is invoked when the customer closes the web chat conversation from the chat widget.
  <br><img width="800" alt="Live Chat Close flow Start node configuration" src="../../../images/LiveChatCloseFlowStartNode.png"><br>
- AliasId is taken in from the Search Conversation response and passed on to the Close Task node as the task identifier, to terminate the task in the backend.
  <br><img width="800" alt="Close Task Node" src="../../../images/CloseTaskNodeInLiveChatCloseFlow.png"><br>

### Configured Web Chat Flow Reference

The supplied `WebChatInboundFlow` follows the standard web chat flow pattern:

`Configure APP Event` → `Search Conversation` → `Pre-chat form` → `Receive` → `Evaluate` → `Resolve Conversation` → `Append Conversation` → `Queue Task` → `Queued`

This flow also supports proactive web chat requests, where the message metadata includes a proactive identifier and queue identifier.

The flow also contains Branch, Close Task, and Error Notification paths. The exact configuration of those supporting nodes should be verified separately before deployment.

#### 1. Configure APP Event

The Start node triggers the flow for an incoming application message.

- APP event: `Incoming message`
- Conditions: Enabled
- Condition: `inappmessaging.message notequals typing_indicator`
- Node ID: `2`

The condition prevents typing-indicator events from starting the customer-message flow. The required mobile or web application can be selected before the flow is made live.

#### 2. Search Conversation

Searches for the existing Web Chat conversation.

- Method: `Search Chat`
- Node authentication: `Your authentication profile`
- Channel: `Web Chat`
- Web chat app: `Your web chat app`
- Web chat thread ID: `$(n2.inappmessaging.threadId)`
- Web chat user ID: `$(n2.inappmessaging.userId)`
- Web chat browser fingerprint: `$(n2.inappmessaging.userId)`
- Version: `v2.2`

#### 3. Pre-chat Form

Sends the configured static pre-chat form to the customer.

- Destination type: `UserId`
- Destination: `$(n2.inappmessaging.userId)`
- Message type: `Form`
- Content type: `STATIC`
- Form template: `Your form template name`
- Thread ID: `$(n2.inappmessaging.threadId)`
- Optional correlation, notify URL, callback, and expiry fields: Not configured

#### 4. Receive

Waits for the customer to submit the pre-chat form.

- Maximum timeout: `300` seconds
- Channel event: `Receive Web Chat`
- From thread ID: `$(n2.inappmessaging.threadId)`
- From user ID: `$(n2.inappmessaging.userId)`
- Event name: `Form Response`
- Content type: `STATIC`
- Form template: `Your form template name`

The Receive node has an on-leave Set Variable action:

- `customerName` = `$(<receiveNode>.inappmessaging.formFields.Name)`
- `customerEmail` = `$(<receiveNode>.inappmessaging.formFields.Email)`

These variables are made available for downstream processing and can be displayed on the Agent Desktop. If the form field names change, update the expressions while keeping the standard flow variable names `customerName` and `customerEmail`.

#### 5. Evaluate

The Evaluate node prepares the message, attachment, and security information required by Resolve Conversation.

- Script output: `1`
- Branch name: `success`

The script:

- Extracts the flow portion from `transid` and identifies the flow ID.
- Reads `messageExtras` and parses it as JSON.
- Detects proactive chat requests using `messageExtras.proactive_id`.
- Sets `isProactiveChat = true` when a proactive identifier is present and captures the corresponding proactive rule and queue identifiers.
- Replaces a blank, undefined, or missing message with `No text`.
- Serializes and escapes newline, quote, slash, and control characters in the message.
- Parses the web chat attachment payload and PCI-compliance reason payload.
- Supports the custom variables `inappPayloadObject` and `nonPCIComplianceReasonObject`.
- Represents dropped attachments with their MIME type, filename `[Dropped]`, an empty URL, and `dropped: true`.
- Represents accepted attachments with the content type, filename extracted from the URL, URL, and `dropped: false`.
- Creates an empty JSON attachment array when no attachment is available.
- Defaults a blank `attachmentCount` to `0`.
- Uses `userId` as a fallback for blank `customerEmail` and `customerName`.
- Builds `messageDetails`, `pciDetails`, `malwareScanDetails`, and `securityScanDetails`.
- Adds `proactiveRuleId` to the payload when the request is proactive.
- Serializes the combined object as `detailsJson`.

The required custom variables are:

- `inappPayloadObject` = `inappmessaging.attachment`
- `nonPCIComplianceReasonObject` = `inappmessaging.pciInfo.nonPCIComplianceReason`
- `messageExtras` = `inappmessaging.extras` or the equivalent JSON payload containing proactive metadata

Proactive handling notes:

- The inbound payload may include a `proactive_id` value when the conversation is initiated by a proactive campaign or rule.
- When `proactive_id` is present and not zero, the flow should treat the chat as proactive and capture the associated proactive metadata.
- The flow should also capture the related `proactive_queue_id` so the task can be routed to the correct queue.
- Standard web chat requests continue to follow the normal inbound flow without proactive routing.
- The proactive context is carried into the resolved conversation payload so downstream reporting and routing can identify it correctly.

#### 6. Resolve Conversation

Creates or updates the Webex Contact Center conversation using the prepared details.

- Method: `Resolve Conversation`
- Node authentication: `Wxcc Node Authorization`
- Trans ID: `$(transId)`
- Media type: `Chat`
- Media channel: `Web Chat`
- Details: `$(detailsJson)`
- Override Default Contact Close Event Handling: Enabled
- Version: `v1.3`

The `detailsJson` payload includes the standard message and compliance data, and for proactive conversations it also includes the proactive rule identifier in `messageDetails.proactiveRuleId`.

#### 7. Append Conversation

Appends the customer’s form response to the existing conversation.

- Method: `Append Chat`
- Node authentication: `Your authentication profile`
- Channel: `Web Chat`
- Conversation ID: `$(conversationId)`
- Message mode: `Single message`
- Message Alias ID: `$(<searchNode>.aliasId)`
- Direction: `Inbound`
- Message type: `Web chat form response`
- Form response: `$(<receiveNode>.inappmessaging.formResponse)`
- Timestamp: `$(<receiveNode>.inappmessaging.timestamp)`
- Version: `v2.2`

#### 8. Queue Task

Submits the conversation to Webex Contact Center for agent assignment.

- Method: `Queue task`
- Node authentication: `Wxcc Node Authorization`
- Task ID: `$(taskId)`
- Conversation ID: `$(conversationId)`
- Media type: `Chat`
- Media channel: `Web Chat`
- Queue mode: `Static`
- Queue name: `Your queue name`
- Contact priority: `Your configured priority`
- Queue routing type: `Longest available agent`
- Version: `v1.8`

For proactive chats, verify the queue mapping and route selection using the proactive rule and queue metadata included in the inbound payload.

#### 9. Queued Message

Sends a confirmation message after the task is successfully queued.

- Node type: `In-app messaging`
- Destination type: `UserId`
- Destination: `$(n2.inappmessaging.userId)`
- Message type: `Message`
- Message: `Message successfully queued`
- Thread ID: `$(n2.inappmessaging.threadId)`
- Notification title/text: Not configured

### Web Chat Deployment Notes

- Update the web chat application, authentication profiles, form template, Flow ID, and queue name for the target tenant.
- Keep `customerName` and `customerEmail` as the standard variables even when form field names change.
- Confirm that the Evaluate node produces `detailsJson` before Resolve Conversation executes.
- Verify PCI, malware, security-scan, and attachment scenarios, including dropped and accepted attachments.
- Test success, timeout, invalid-channel, proactive-chat, and error-notification paths before making the flow live.
- Validate the proactive metadata payload so `proactiveRuleId` is set correctly for proactive web chat sessions.

## Enhancements Included

- Chat
  - Support for displaying customer name and emailId if available via form, on the Agent's Desktop
- Bot flows
  - Updated to ensure flows get imported without any errors
