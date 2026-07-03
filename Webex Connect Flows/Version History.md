
## Version History

## v3.5

### What Changed

The v3.5 release focuses on correctness fixes in the Email Inbound Flow.

- **Email Inbound Flow — Attachment MIME type filtering:** The Evaluate Node in the Email Inbound Flow was updated to check each attachment's MIME type before passing it to the Resolve Conversation Node. Attachments with a `message/rfc822` type (raw email message attachments) are now filtered out and excluded from the Resolve Conversation Node call, avoiding an error that occurred when such attachments were passed through.

- **Email Inbound Flow — Inline image CID forwarding:** The flow was updated to extract and forward the `cid` (Content-ID) parameter from inline image attachments to the Engage API. This ensures that inline images embedded within email bodies are correctly referenced and rendered in the conversation.

### Steps to Upgrade

Refer to [How to manually upgrade from v3.4 to v3.5 workflows](./How%20to%20manually%20upgrade%20from%20v3.4%20to%20v3.5%20workflows.MD)


## v3.4

### What Changed

The v3.4 release updates all flows to use the latest available node versions and adds support for proactive chat in the Live Chat Inbound Flow.

- **Node version updates:** All flows were reviewed and updated to reference the latest versions of each node type, ensuring compatibility with current Webex Connect platform capabilities.

- **Live Chat Inbound Flow — Proactive chat support:** A new variant of the Live Chat Inbound Flow was introduced that supports proactive chat messages. This allows the contact center to initiate a chat interaction with a customer rather than waiting for an inbound message.

- **Media Specific Flows:** The email inbound flow received the latest node version updates. The remaining digital channel flows were simplified to reduce complexity while maintaining full functionality.

### Steps to Upgrade

Refer to [How to manually upgrade from v3.3 to v3.4 workflows](How%20to%20manually%20upgrade%20from%20v3.3%20to%20v3.4%20workflows.MD)


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


## v3.2

### What Changed

The v3.2 release introduces contact priority support and a minor email sender address improvement.

- **Contact Priority:** The Queue Task Node was updated to accept a contact priority value. Flow administrators can now set the priority of a contact when it is submitted to the queue, allowing higher-priority contacts to be routed to agents before lower-priority ones. The Usage of Contact Priority sample flow demonstrates this via an Email Flow.

- **Email sender address:** Flows were updated to support receiving emails from flows where the sending address matches the asset's configured email address, improving routing accuracy for email-based workflows.

### Steps to Upgrade

Refer to [How to manually upgrade from v3.1 to v3.2 workflows](How%20to%20manually%20upgrade%20from%20v3.1%20to%20v3.2%20workflows.MD)


## v3.1

### What Changed

The v3.1 release adds malware scanning for attachments and several channel-specific improvements.

- **Malware scanning — Channel Start Node enhancements:** The Channel Start Nodes for all supported channels were updated to expose a new set of output variables that carry the results of the platform's built-in malware scan. These variables indicate whether an attachment was scanned and what the security verdict was. Flow administrators can use a Branch Node after the Channel Start Node to check these values and route unsafe attachments down an error path rather than processing them further. Only attachments that pass the security scan proceed down the success path.

- **Live Chat / Chat:** The flow was updated so that `customerName` and `customerEmail` values collected via the pre-chat form are correctly passed through the Receive Node Transition Actions and surfaced on the Agent Desktop.

- **Email:** Three edge cases were addressed — emails sent without a subject line, emails sent without a sender name, and plain-text emails (with no HTML body) are now handled without errors.

- **Bot flows:** Minor import errors present in v3.0 bot flows were resolved so flows can be imported cleanly.

### Steps to Upgrade

Refer to [How to manually upgrade from v3.0 to v3.1 workflows](How%20to%20manually%20upgrade%20from%20v3.0%20to%20v3.1%20workflows.MD)



## v3.0

### What Changed

The v3.0 release is a foundational simplification of the overall flow architecture.

- **Resolve Conversation Node — Simplified flow logic:** Contact Center–specific backend logic that previously required multiple nodes and manual configuration steps in each flow was consolidated into a single **Resolve Conversation Node**. This node handles the full termination lifecycle of a conversation (closing the task, resolving the conversation record, and performing any required backend callbacks) with minimal configuration. Flow administrators only need to provide the Flow ID to the node.

- **Optional Event Handling Flows:** The shared Task Routed, Task Modified, and Task Closed flows are now fully optional. Webex Contact Center provides default built-in handling for all task lifecycle events. Flow administrators only need to deploy event flows if they require custom business logic (such as Screen Pop or HTTP API calls) on top of the default behaviour.

- **Region-agnostic flows:** All flows were redesigned to be deployment region agnostic, removing hardcoded region-specific endpoints and allowing the same flow to be used across any Webex Contact Center deployment region.
