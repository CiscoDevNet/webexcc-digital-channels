# How to manually upgrade from v3.5 to v3.6 workflows

In case you have made custom changes to your inbound flows, you can manually upgrade your existing v3.5 flows to v3.6. This ensures that the custom changes are retained and do not have to be repeated. Follow the steps below for the same.

## Overview of the v3.6 changes

The v3.6 release simplifies the template flows and standardizes them around the single Resolve Conversation node. This node handles the Contact Center-specific backend resolution logic with minimal manual configuration.

This means:

- the template flow structure is simplified and standardized
- the Evaluate node prepares the payload before the Resolve Conversation step
- the Resolve Conversation node is the key step for resolving the conversation
- custom logic from the v3.5 flow should be retained while adapting to the v3.6 structure

## Recommended upgrade approach

1. Take a backup of the current v3.5 flow.
2. Import or recreate the corresponding v3.6 template flow.
3. Compare the old flow and the v3.6 template and keep only the required custom branches or variables.
4. Update the Resolve Conversation node with the correct Flow ID for your tenant.
5. Validate that the Evaluate node generates the required `detailsJson` payload correctly.
6. Ensure custom branching, queueing, routing, or event handling logic remains intact.
7. Test the flow with a real inbound message to confirm the upgrade works as expected.

## For inbound flows with custom changes

- Upgrade the flow to the v3.6 template structure.
- Review the updated Evaluate logic and confirm that the payload passed to Resolve Conversation is correct.
- Preserve any custom sender, subject, routing, queue, or attachment-related logic from the old flow.
- Ensure the Resolve Conversation node is configured with the correct Flow ID and latest payload format.

> Note: The main change in v3.6 is the simplification of the flow architecture. Do not overwrite custom logic blindly; compare the old flow with the v3.6 template and migrate only the required updates while keeping the custom behavior intact.
