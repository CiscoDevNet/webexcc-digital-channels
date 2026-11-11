# WhatsApp Inbound Flow

## Overview

This README describes the WhatsApp inbound flow used to process customer messages received through a configured WhatsApp Business number.

The flow is responsible for:

- receiving the inbound WhatsApp message
- mapping the message and related metadata
- creating or resolving the Webex Contact Center conversation
- sending the required WhatsApp response or notification
- queueing the task for agent handling
- handling timeout, policy, and error paths

This document is intentionally limited to WhatsApp-related content and does not include other channel workflows.

## Flow Sequence

`Configure WhatsApp Event` → `Evaluate` → `Resolve Conversation` → `WhatsApp (Message is resolved)` → `Queue Task` → `WhatsApp (Message successfully queued)`

Additional paths may include Branch, Close Task, timeout, policy-failure, and error-notification handling depending on the tenant configuration.

## Prerequisites

Before making the flow live:

- select the correct WhatsApp asset for the tenant
- update the custom variables in the flow settings
- configure the correct authorization profile for the integration
- select the correct queue for WhatsApp task routing
- verify the Flow ID and node references in your environment, as these vary by tenant

## Updating Flow Variables

Update the custom variables in the flow settings for your environment.

Typical WhatsApp-related variables include:

- `messengerPayloadObject`
- `messagetext`
- `message`
- `attachmentURL`
- `nonPCIComplianceReasonObject`
- `isPCIValidationDone`
- `owner`
- `customerUsername`
- `appid`
- `customerWhatsappId`
- `caption`
- `serviceKey`
- `timestamp`
- `transId`
- `detailsJson`

The exact variable names and values depend on the Webex Connect flow configuration in your tenant.

## 1. Configure WhatsApp Event

This Start node triggers when an inbound WhatsApp message is received.

Typical configuration:

- Channel: `WhatsApp`
- Event: `Incoming message`
- Conditions: `Not enabled` unless specific filtering is required
- Node authentication: `Your configured authorization profile`

The transition action maps the inbound payload into variables used by the Evaluate node, such as:

- `messengerPayloadObject` = `$(n2.whatsapp.attachments)`
- `messagetext` and `message` = `$(n2.whatsapp.message)`
- `attachmentURL` = `$(n2.whatsapp.attachments)`
- `nonPCIComplianceReasonObject` = `$(n2.whatsapp.pciInfo.nonPCIComplianceReason)`
- `isPCIValidationDone` = `$(n2.whatsapp.pciInfo.isPCIValidationDone)`
- `customerUsername` = `$(n2.whatsapp.username)`
- `appid` = `$(n2.whatsapp.appId)`
- `customerWhatsappId` = `$(n2.whatsapp.waId)`
- `caption` = `$(n2.whatsapp.caption)`
- PCI, malware, and security scan variables from the corresponding WhatsApp payload fields

## 2. Evaluate

The Evaluate node prepares the WhatsApp message, attachment, and security information required by Resolve Conversation.

The script typically:

- reads the inbound WhatsApp payload
- normalizes the message text and timestamp values
- parses the attachment payload from `messengerPayloadObject`
- creates a normalized attachment array with MIME type, filename, URL, and `dropped` status
- marks PCI-rejected attachments as `dropped: true`
- marks accepted attachments as `dropped: false`
- builds PCI, malware, and security scan details
- serializes the final payload as `detailsJson`

The output is sent on the `success` branch when the payload is successfully prepared.

## 3. Resolve Conversation

This node creates or updates the Webex Contact Center conversation for the inbound WhatsApp interaction.

Typical settings:

- Method: `Resolve Conversation`
- Node authentication: `Your configured authorization profile`
- Flow ID: `Use the Flow ID configured for your tenant`
- Trans ID: `$(transId)`
- Media type: `Social`
- Media channel: `WhatsApp`
- Details: `$(detailsJson)`
- Override Default Contact Close Event Handling: `As configured for your environment`
- Version: `As per the flow version in your tenant`

The `detailsJson` value contains the message, attachments, compliance data, and scan details required for agent handling.

## 4. WhatsApp Resolution Message

After the conversation is resolved, the WhatsApp node sends a confirmation to the customer.

Typical configuration:

- Destination type: `WA ID`
- Destination: `$(customerWhatsappId)` or equivalent variable from the inbound payload
- Message type: `TEXT`
- Message: `Message is resolved.`
- Enable Preview URL: `As configured`
- Optional correlation ID, notify URL, and callback data: `Tenant-specific`

This confirms that the customer message was accepted and the conversation is being handled.

## 5. Queue Task

This node submits the WhatsApp interaction to Webex Contact Center for agent assignment.

Typical settings:

- Method: `Queue task`
- Node authentication: `Your configured authorization profile`
- Task ID: `$(taskId)`
- Conversation ID: `$(conversationId)`
- Media type: `Social`
- Media channel: `WhatsApp`
- Queue mode: `Static`
- Queue name: `Your configured WhatsApp queue`
- Contact priority: `As configured for your environment`
- Queue routing type: `Longest available agent`
- Version: `As per the flow version in your tenant`

The queue selection is mandatory before making the flow live.

## 6. Queued Message and Error Handling

After the task is queued, the flow may continue with a queued notification or branch-based handling.

Typical behavior:

- confirm successful queueing
- send a queued WhatsApp message to the customer
- handle timeout, policy-failure, invalid-choice, and service-unavailable paths
- route error notifications appropriately

The exact message text and branch handling should be validated in the tenant flow before deployment.

## WhatsApp Deployment Notes

- Update the WhatsApp number, application, authentication profile, Flow ID, and queue name for the target tenant.
- Confirm that all custom variables are populated correctly from the WhatsApp payload.
- Confirm that `detailsJson` is generated before `Resolve Conversation` executes.
- Validate normal messages, blank messages, attachments, dropped attachments, PCI outcomes, malware/security scan results, queue success, and error paths.
- Verify the final flow in the target environment because node IDs and authorization profiles vary by tenant.

## Best Practices

- Keep the flow limited to WhatsApp-specific logic only.
- Use generic placeholders instead of tenant-specific values in documentation.
- Verify the queue, authorization profile, and Flow ID in the target environment before moving to production.
- Validate the final tenant flow for both happy-path and failure-path scenarios.

## Related References

For implementation guidance, refer to the Webex Connect platform documentation and the tenant-specific flow configuration used in your environment.
