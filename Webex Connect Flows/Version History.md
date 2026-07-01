# Version History

## v3.5

#### Features released:

Email Attachment Evaluate node fix for avoid passing on message/rfc822 attachments to the Resolve conversation node limitation on Webex Connect's Email.

Email inbound flow: Changes made to forward cid parameter to engage-api for inline image attachments

## v3.4

#### Features released:

Latest version nodes updated in flows

## v3.3

#### Features released:

- Capability to deliver system messages such as survey links and survey messages to customers post contact handling
- Apple Business Messages
  - Support for Apple Business Messages as a channel

## v3.2

#### Features released:

- Option to set contact priority while queueing contacts

#### Minor updates:

- Support for receiving emails from flows with the asset's email as the sender's address

## v3.1

#### Features released:

- Malware scanning for attachments
  - Channel start nodes have been improved to include a new set of variables for identifying malware and presenting security scan results
  - Only safe attachments will be processed further

#### Minor updates:

- Chat
  - Support for displaying customer name and emailId if available via form, on the Agent's Desktop
- Email
  - Support for emails sent without subject
  - Support for emails sent without sender name
  - Support for plain text emails
- Bot flows
  - Fixed minor issues in flows during import

## v3.0

#### Features released:

- Simplified Flows
  - Replacement of contact center specific logic in flows with the new Resolve Conversation node
  - Shared flows like Task Routed, Task Modified and Task Closed simplified and made optional
