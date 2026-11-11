# Apple Messages for Business Inbound Flow

## Overview

This README describes the Apple Messages for Business (AMB) inbound flow used to process customer messages received through an integrated Apple Business account.

The flow is responsible for:

- receiving the inbound AMB message
- mapping the message and associated compliance metadata
- resolving the Webex Contact Center conversation
- sending the required AMB response or confirmation
- queueing the task for agent handling
- handling timeout, policy, and error paths

## Flow Sequence

`Configure Apple Messages for Business Event` → `Evaluate` → `Resolve Conversation` → `Apple Messages for Business (Message is resolved)` → `Queue Task` → `PIQ and EWT` → `Apple Messages for Business (Message is queued)`

Additional paths may include Branch, Close Task, timeout, policy-failure, and error handling depending on the tenant configuration.

## Prerequisites

Before making the flow live:

- select the correct Apple Messages for Business asset for the tenant
- update the custom variables in the flow settings
- configure the correct authorization profile for the integration
- select the correct queue for AMB task routing
- verify the Flow ID and node references in your environment, as these vary by tenant

## Updating Flow Variables

Update the following custom variables in the flow settings for your environment, as required by the AMB flow:

- `abcUserId`
- `attachments`
- `attachmentCount`
- `attachmentUrl`
- `accountId`
- `appId`
- `timestamp`
- `transId`
- `serviceKey`
- `isPCICompliance`
- `isPCIValidationDone`
- `nonPCIComplianceReasonObject`
- `isAttachmentEnabled`
- `droppedAttachmentCount`
- `detailsJson`

The exact names and values may vary slightly based on the tenant configuration and node mapping in Webex Connect.

## 1. Configure Apple Messages for Business Event

The Start node triggers the flow for an incoming Apple Messages for Business message.

Typical configuration:

- Channel: `Apple Messages for Business`
- Event: `Incoming message`
- Node authentication: `Your configured authorization profile`

The transition action maps the inbound Apple Business Chat payload into flow variables, such as:

- `abcUserId`
- `attachments`
- `attachmentCount`
- `attachmentUrl`
- `accountId`
- `appId`
- `timestamp`
- `transId`
- `serviceKey`
- `resolveConversationmessagetext`
- `resolveConversationIsPCIValidationDone`
- `resolveConversationNonPCIComplianceReasonObject`
- `isAttachmentEnabled`
- `droppedAttachmentCount`
- `isPCICompliance`

These values are used by the Evaluate node to create the conversation payload for Webex Contact Center.

## 2. Evaluate

The Evaluate node prepares the AMB message, attachment, and compliance information required by Resolve Conversation.

Typical script behavior:

- reads the inbound Apple Business message payload
- extracts the transaction ID from `transId`
- builds `messageDetails` with the app ID, user ID, account ID, message, attachments, service key, and timestamp
- converts `NA` PCI values into the expected JSON-friendly format before parsing
- includes PCI validation status, non-PCI compliance details, attachment enablement, and dropped attachment count
- combines the message and PCI objects into a single payload and serializes it as `detailsJson`

The output is sent on the `success` branch when the payload is prepared correctly.

## 3. Resolve Conversation

This node creates or updates the Webex Contact Center conversation for the inbound AMB interaction.

Typical settings:

- Method: `Resolve Conversation`
- Node authentication: `Your configured authorization profile`
- Flow ID: `Use the Flow ID configured for your tenant`
- Trans ID: `$(transId)`
- Media type: `Social`
- Media channel: `Apple Messages for Business`
- Details: `$(detailsJson)`
- Override Default Contact Close Event Handling: `As configured for your environment`
- Version: `As per the flow version in your tenant`

The `detailsJson` payload includes the AMB message, attachment metadata, and compliance details required for downstream routing and agent handling.

## 4. AMB Resolution Message

After the conversation is resolved, the AMB node sends a response to the customer.

Typical configuration:

- Destination type: `AbcUserId`
- Destination: `$(abcUserId)` or equivalent variable from the trigger payload
- Message type: `Text` or `Text With Attachments`
- Message: `Message is resolved`
- Send typing indicator: `As configured`
- Correlation ID, notify URL, callback data, and expiry: `Optional and tenant-specific`

This step confirms to the customer that the message was accepted and the conversation is being handled.

## 5. Queue Task

This node submits the AMB conversation to Webex Contact Center for agent assignment.

Typical settings:

- Method: `Queue task`
- Node authentication: `Your configured authorization profile`
- Task ID: `$(taskId)`
- Conversation ID: `$(conversationId)`
- Media type: `Social`
- Media channel: `Apple Messages for Business`
- Queue mode: `Static`
- Queue name: `Your configured AMB queue`
- Contact priority: `As configured for your environment`
- Queue routing type: `Longest available agent`
- Version: `As per the flow version in your tenant`

Queue selection is mandatory before making the flow live.

## 6. PIQ/EWT and Queued Confirmation

After the task is queued, the flow may route through PIQ/EWT processing and then send the queued confirmation message.

Typical behavior:

- evaluate queue wait and routing status
- confirm that the task has been queued successfully
- send a final AMB confirmation to the customer

The exact PIQ/EWT node configuration and queued message content should be verified inside the tenant flow before deployment.

## 7. Close Task and Supporting Paths

The AMB flow may also include:

- Branch logic
- Close Task handling
- timeout handling
- policy-failure handling
- error-notification paths

These supporting nodes should be verified separately in the tenant flow before making the flow live.

## AMB Deployment Notes

- Update the Apple Messages for Business application, authentication profile, Flow ID, and queue name for the target tenant.
- Confirm that the required inbound variables are populated correctly from the AMB event payload.
- Confirm that `detailsJson` is generated before `Resolve Conversation` executes.
- Validate text messages, attachments, PCI outcomes, security scan results, queue success, timeout handling, and close-task behavior.
- Verify the final flow in the target environment because node IDs and authorization profile names vary by tenant.

## Best Practices

- Keep the flow limited to Apple Messages for Business logic only.
- Use generic placeholders instead of tenant-specific values in documentation.
- Verify the queue, authorization profile, and Flow ID in the target environment before moving to production.
- Validate the final tenant flow for all happy-path and failure-path scenarios.

## Related References

For implementation guidance, refer to the Webex Connect platform documentation and the tenant-specific flow configuration used in your environment.
