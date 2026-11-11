# SMS Inbound Flow

## Overview

This README describes the SMS inbound flow used to process customer messages received through a configured SMS number.

The flow is responsible for:

- receiving the inbound SMS message
- mapping the message and related metadata
- creating or resolving the Webex Contact Center conversation
- sending the required SMS response or notification
- queueing the task for agent handling
- handling timeout and error paths

This document is intentionally limited to SMS-related content and does not include other channel workflows.

## Flow Sequence

`Configure SMS Event` → `Evaluate` → `Resolve Conversation` → `SMS Notification` → `Queue Task` → `Queued`

Additional paths may include Branch, Close Task, timeout, and error-notification handling depending on the tenant configuration.

## Prerequisites

Before making the flow live:

- select the correct SMS asset for the tenant
- update the custom variables in the flow settings
- configure the correct authorization profile for the integration
- select the correct queue for SMS task routing
- verify the Flow ID and node references in your environment, as these vary by tenant

## Updating Flow Variables

Update the custom variables in the flow settings for your environment.

Typical SMS-related variables include:

- `serviceNumber`
- `senderNumber`
- `serviceKey`
- `timestamp`
- `message`
- `transId`
- `isPCICompliance`
- `isPCIValidationDone`
- `nonPCIComplianceReasonObject`
- `securityScanDetails`
- `malwareScanDetails`
- `detailsJson`

The exact variable names and values depend on the Webex Connect flow configuration in your tenant.

## 1. Configure SMS Event

This start node triggers when an inbound SMS message is received.

Typical configuration:

- Channel: `SMS`
- Event: `Mobile Originated (MO)`
- Keyword: `*` or as required by the use case
- Conditions: `Not enabled` unless specific filtering is required
- Node authentication: `Your configured authorization profile`

The transition action maps the inbound message fields into variables that are used by the Evaluate node, such as:

- `serviceNumber`
- `senderNumber`
- `message`
- `serviceKey`
- `timestamp`
- `transId`
- compliance and security metadata

## 2. Evaluate

The Evaluate node prepares the payload required by the Resolve Conversation node.

The script typically:

- reads the inbound SMS details from the trigger payload
- normalizes values such as message text and timestamps
- converts PCI, malware, and security validation data into the required JSON format
- builds a final `detailsJson` payload for the conversation object
- marks empty or invalid values safely for downstream processing

The output is sent to the `success` branch when the payload is successfully prepared.

## 3. Resolve Conversation

This node creates or updates the Webex Contact Center conversation for the inbound SMS interaction.

Typical settings:

- Method: `Resolve Conversation`
- Node authentication: `Your configured authorization profile`
- Flow ID: `Use the flow ID configured for your tenant`
- Trans ID: `$(transId)`
- Media type: `Social`
- Media channel: `SMS`
- Details: `$(detailsJson)`
- Override Default Contact Close Event Handling: `As configured for your environment`
- Version: `As per the flow version in your tenant`

The `detailsJson` value contains the parsed customer SMS data, compliance results, and payload details required for agent handling.

## 4. SMS Notification

This node sends the customer a response or notification through SMS.

Typical configuration:

- Destination type: `MSISDN`
- Destination: `$(senderNumber)` or equivalent variable from the inbound message
- From number: `Your configured SMS sender number`
- Message type: `Text`
- Message: `Use the relevant success or error text configured for your flow`
- Shorten links: `As configured`

This node may be used for either a success response or an error notification depending on the flow path.

## 5. Queue Task

This node submits the SMS interaction to Webex Contact Center for agent assignment.

Typical settings:

- Method: `Queue task`
- Node authentication: `Your configured authorization profile`
- Task ID: `$(taskId)`
- Conversation ID: `$(conversationId)`
- Media type: `Social`
- Media channel: `SMS`
- Queue mode: `Static`
- Queue name: `Your configured SMS queue`
- Contact priority: `As configured for your environment`
- Queue routing type: `Longest available agent`
- Version: `As per the flow version in your tenant`

The queue selection is mandatory before making the flow live.

## 6. Queued and Error Handling

After the task is queued, the flow may continue with queued notifications or branch-based handling.

The flow may also include:

- queue timeout handling
- validation or security failure handling
- error notification by SMS
- close-task processing
- branch logic for alternate routes

These paths should be verified in the tenant flow before production deployment.

## SMS Deployment Notes

- Update the SMS sender number and queue configuration for the target tenant.
- Confirm that all custom variables are populated correctly from the inbound SMS payload.
- Ensure `detailsJson` is generated before `Resolve Conversation` executes.
- Validate normal SMS messages, blank messages, PCI validation results, malware/security scan outcomes, queue timeout handling, and error cases.
- Verify the final flow in the target environment because node IDs and authorization profile names vary by tenant.

## Best Practices

- Keep the flow limited to SMS-specific logic only.
- Use generic placeholders rather than tenant-specific values in documentation.
- Validate the flow in the target environment before moving it to production.
- Confirm the queue, sender number, and authorization profile are configured correctly for the environment.

## Related References

For implementation guidance, refer to the Webex Connect platform documentation and the tenant-specific flow configuration used in your environment.
