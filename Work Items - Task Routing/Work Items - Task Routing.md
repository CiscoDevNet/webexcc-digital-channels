---

title: Task routing - Work Item

display_title: Task routing - Work Item

url: /docs/work-items

navigation_category:

- webex-contact-center

product_group:

- webex-contact-center

search_category: Documentation

---
# Work Items - Task routing

Work Items enable external systems to inject **non-conversational tasks** into Webex Contact Center and can be orchestrated within Webex CC Flow Designer, routed, handled, monitored, and reported alongside other contact center channel interactions.

**Note:** 
- Work Items do NOT represent live messaging conversations for unsupported channels such as Viber, Instagram, third-party digital channel gateways, etc. Bring your own messaging channel should be used for such use-cases.
- Use Work Items specifically for backend or offline tasks such as CRM leads, claims, faxes, cases, purchase orders, or other work that needs contact center routing and agent handling without a customer messaging on the other side.

Availability for Work Items is currently open for **BETA**. Contact your Cisco team to request access.

# How does this work

- Developers are expected to host an external system (a middleware) that owns the source task and work item metadata. This is expected to be developer by the partner or customer that integrates with upstream systems and injects tasks into Webex Contact Center.
- Webex Contact Center owns routing, flows, agent handling, task state, lifecycle events, and reporting
- Administrators configure the following entities on **Control Hub** at a high level:
	- **Work Item Channel:** Represents the custom channel that you plan to integrate with (e.g. Fax)
	- **Work Item Asset:** Represents an instance of the channel added above and establishes the data schema payloads you wish to honour when injecting tasks into Webex Contact Center
	- **Flow:** Represents the flow that manages orchestrating a work item and routing it to queues
	- **Queue:** Queues now support a new channel type: `Work Item` 
	- **Multimedia Profile:** Determines the no. of tasks an agent can work on in parallel either via automatic contact distribution or self-assignment (cherry picking)
- Developers use **Public APIs** for the following:
	- **Webhook events:** Subscribe to task lifecycle and message events using **Subscriptions API**
	- **Create work item:** Create a work item using **Create Task API v2**
	- **Update work item:** Update a work item's data using **Task Messages API**
	- **Search for work items:** To look up existing work items using **Search API**
	- **Retrieve transcripts:** Pull post interaction transcripts using **Captures API**

## Responsibilities

### External developer

Your **middleware** will be mainly responsible for creating or updating Work Item tasks in Webex Contact Center. Below are the requirements that the developer is expected to manage:

- Hosting the middleware that receives or produces the external work item
- Obtaining and refreshing OAuth tokens to integrate with Webex CC's Task API
- Integration with Webex CC APIs as required - Tasks, Subscription, Search, Captures
- Ensuring that the `Work Item Data`conforms to the administrator-configured Work Item schema on the Asset configured on Control Hub
### Webex Contact Center

Webex Contact Center is responsible for:

- Validating Work Item task requests against configured assets, entry points, and schema
- Running the configured Work Item flow as configured against the Entry point
- Routing Work Items to queues and agents as per the configuration on Webex CC Flow Designer
- Presenting Work Item metadata to agents on the Desktop as read-only data
- Generating task lifecycle and Work Item update webhook events
- Exposing post interaction transcripts for Work Item containing snapshots of the data held within Webex CC.

## Before You Begin

Before integrating, rensure that the administrator has configured the following in Contol Hub:

- A `Work Item Custom Channel` with:
	- `Custom channel name`
	- `Logo` (optional)
- A `Work Item Asset` with:
	- `Asset name` and `Description` (optional)
	- `Business address` - Work Item assets use a configured business address to resolve Create Task requests. The business address must be unique across Work Item assets.
	- `Data schema fields` - Work Item schema configuration defines which metadata fields are accepted for the task. The configured schema can contain up to 50 fields. Each field key can contain up to 100 characters, and each field value can contain up to 1024 characters. At least one required field and one viewable field must be configured before a Work Item can be saved. These policies are enforced in the Create Task and Task Messages API. A schema field includes:
		- `Field key`
		- `Display name`
		- Whether the field is `required`
		- Whether the field is `viewable`
		  **Note:** Atleast 1 viewable and 1 required field is **mandatory**
- A `Work Item Entry Point` mapped to a `Work Item Asset` and a `Work Item Flow`
- `Work Item Queues` and queue ranking within relevant `Teams` as needed
- `Multimedia profile` capacity for Work Items. Please note that we support both **automatic-assignment** and **self-assignment** (cherry picking) similar to other digital channels.
- `RONA timeout` values for Work Items as needed
- Other Contact Center routing objects required for the interaction, such as teams, skills, queues, and agent profiles

## Authenticating APIs

Work Item injection leverages a [Service App](/docs/service-apps) that will represent your middleware so that `access_tokens` handling are managed appropriately for server-side operations.

For `scope` definitions, refer to the [Integrations guide](/docs/api/guides/integrations).

### Create and authorise a service app

1. Sign in to the developer portal and create a new **Service App**
2. Capture the client secret when it is first shown. After the app is submitted for authorization, regenerate the secret if it was not saved.
3. Assign the required scopes:
	- `cjp:task_write`
	- `cjp:task_read`
4. Ask the customer administrator to authorise the Service App in Control Hub under **Apps**.
5. After authorization, return to the Service App in the developer portal, select the authorised organization, provide the client secret, and generate the access token.
6. Use the resulting access token when calling the Create Task API and the Task Messages API.

For this integration pattern, ensure that:

- The Service App is authorized for the customer organization
- The `access_token` includes `cjp:task_write` for Create Task and Task Messages
- The `access_token` includes `cjp:task_read` when task read access is required

### Refreshing access tokens  

If your integration needs long-running access, retain the refresh token and use the standard OAuth refresh flow described in the [Authentication guide](/docs/authentication-cc) before the current access token expires.

For server-to-server integrations, store the `client_secret`, `access_token`, and `refresh_ token` securely, and rotate or regenerate credentials according to your organization's security practices.
## Work Item schema behaviour

As mentioned previously, Work Item schema is configured on the Work Item asset. The schema controls the metadata fields that can be supplied when creating or updating a Work Item.

For Work Item API requests:
- `workItemData` must contain at least one field
- Field keys must match keys configured in the Work Item asset schema
- Required schema fields must be present and non-empty
- Required fields sent with `null` are treated as empty
- Values are validated against configured schema constraints
- Invalid, missing, unknown, or empty required fields can cause the request to fail

**Note:**  
- Work Items do NOT support attachments currently.
- They can only be supported in inbound direction
## Subscribe to task and message events

Use the [Subscriptions API](/webex-contact-center/docs/api/v2/subscriptions) for task lifecycle and Work Item update webhook events.

Important event families for Work Item integrations include the below at the very least as these will allow you to stay informed if there was an error creating a work item

- `task:new`
- `task:failed`
- `task-message:appended`
- `task-message:append-failed`

## Create a new work item

Use the [Create Task API](/webex-contact-center/docs/api/v2/tasks-call-control/create-task) to create the initial Work Item task.

**Example request:**

```json
{
	"origin": {
		"id": "<origin-identifier>",
		"name": "<customer-name>"
	},
	"destination": {
		"id": "<your-biz-address>",
		"type": "businessAddress"
	},
	"channelType": "workItem",
	"channel": "<work-item-custom-channel-name>",
	"flowSettings": {
		"foo": "bar"
	},
	"globalVariables": {
		"var1": "val1"
	},
	"channelParams": {
		"type": "work-item-form",
		"message": {
			"aliasId": "51b8f31d-05b2-45f8-b172-63920f2b7d9a",
			"workItemData": {
				"customerAccountId": "3572653863",
				"customerAccountName": "Customer Name",
				"claimNumber": "CLM1234567",
				"priority": "High"
			},
			"timestamp": 1779785508649
		}
	}
}
```

**Important request fields:**

| Parameter                            | Description                                                                            | Mandatory |
| ------------------------------------ | -------------------------------------------------------------------------------------- | --------- |
| `origin.id`                          | Developer-defined customer / case / task, or source-system identifier                  | Yes       |
| `origin.name`                        | Display name associated with the Work Item origin. Will appear as the ANI in reporting | No        |
| `destination.id`                     | Business address configured on the Work Item asset                                     | Yes       |
| `destination.type`                   | Must be `businessAddress`                                                              | Yes       |
| `channelType`                        | Must be `workItem`                                                                     | Yes       |
| `channel`                            | Configured `Work Item Channel Name`                                                    | Yes       |
| `flowSettings`                       | Flow variables supplied to the task                                                    | No        |
| `globalVariables`                    | Global variables mapped to the task                                                    | No        |
| `channelParams.type`                 | Must be `work-item-form`                                                               | Yes       |
| `channelParams.message.aliasId`      | Developer-provided Work Item data identifier for correlation. Set it to a UUID         | Yes       |
| `channelParams.message.workItemData` | Metadata fields that conform to the Work Item asset schema                             | Yes       |
| `channelParams.message.timestamp`    | Work Item timestamp in epoch milliseconds                                              | Yes       |

The guide shows one representative Work Item example. For the full set of supported request combinations and validation rules, refer to the [Create Task API](/webex-contact-center/docs/api/v2/tasks-call-control/create-task).

**Example response:**

```json

{
	"meta": {
		"orgId": "cb5901fb-ab66-4377-a7bc-5f0b21896952"
	},
	"data": {
		"id": "141897a0-6212-49be-be9d-4823ab1e4dc7"
	}
}

```

Use `data.id` as the created task identifier. You should retain this value for subsequent Task Messages API calls and to correlate later task lifecycle subscription webhooks for the interaction.
## Update Work Item Data

Use the [Task Messages API](/webex-contact-center/docs/api/v2/tasks-call-control/append-task-message) to append updated a subsequent snapshot of the Work Item data to the same task.

For Work Items, this API appends a metadata update. It does not append a conversational chat message.

**Example request:**
```json
{
	"channelParams": {
		"type": "work-item-form",
		"message": {
			"type": "WorkItemForm",
			"aliasId": "991d52ea-927f-41ff-832b-d8f8601a8fbd",
			"workItemData": {
				"customerAccountId": "3572653863",
				"customerAccountName": "Updated Customer Name",
				"claimStatus": "Approved"
			},
			"timestamp": 1779789108649
		}
	}
}
```

**Example response:**

```json

{
	"meta": {
		"orgId": "cb5901fb-ab66-4377-a7bc-5f0b21896952"
	},
	"data": {
		"id": "991d52ea-927f-41ff-832b-d8f8601a8fbd"
	}
}

```

- Use `data.id` as the update event correlation identifier. If request `aliasId` is a valid UUID, the platform can reuse that value as the event identifier.
- After a successful update, Webex Contact Center publishes the inbound `task-message:appended` subscription webhook for the same Work Item update, and the detailed event payload is documented in [Webhook Event Details](/docs/api/guides/webhooks-cc).

**Important request fields:**

- `channelParams.type`: Must be `work-item-form`
- `channelParams.message.aliasId`: Developer -provided Work Item update identifier for correlation
- `channelParams.message.workItemData`: Metadata fields to append to the Work Item transcript
- `channelParams.message.timestamp`: Work Item update timestamp in epoch milliseconds

**Common failures**

Common Work Item update failure categories surfaced through `task-message:append-failed` are shown below, and the detailed event payload is documented in [Webhook Event Details](/docs/api/guides/webhooks-cc):

| Failure reason    | Typical meaning                                                                                                               | Typical next step                                |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| `INVALID_CONTENT` | The request payload violates Work Item schema or validation rules, such as missing required fields or invalid `workItemData`. | Correct the request payload and retry.           |
| `NOT_FOUND`       | The referenced task context could not be found.                                                                               | Verify the task identifier before retrying.      |
| `INTERNAL_ERROR`  | The platform could not complete the update because of an unexpected internal failure.                                         | Investigate the related failure event and retry. |
  
## Webhook events

  Use the [Subscriptions API](/webex-contact-center/docs/api/v2/subscriptions) for task lifecycle and Work Item update webhook events.

Important event families for Work Item integrations include:

- `task:new`
- `task:failed`
- `task-message:appended`
- `task-message:append-failed`

Use the [List Event Types API](/webex-contact-center/docs/api/v2/subscriptions/list-event-types) to discover the latest supported event types and resource versions.

Use the [Webhooks guide](/docs/api/guides/webhooks-cc) for detailed task and task-message event payload documentation.

For Work Items, subscription-based `task-message:appended` and `task-message:append-failed` describe Work Item metadata updates. They do not represent customer chat messages.

**Representative work item `task-message:appended` subscription webhook**

```json
{
	"id": "4c2c4890-8a36-4dcc-9702-4c3a4aa1dc51",
	"specversion": "1.0",
	"type": "task-message:appended",
	"source": "/com/cisco/wxcc/ed9357d0-3f5d-4c6a-a4fd-111111111111",
	"comciscoorgid": "cb5901fb-ab66-4377-a7bc-5f0b21896952",
	"comciscotimestamp": 1779789108649,
	"datacontenttype": "application/json",
	"data": {
		"taskId": "141897a0-6212-49be-be9d-4823ab1e4dc7",
		"direction": "INBOUND",
		"messageDirection": "INBOUND",
		"channelType": "workItem",
		"channel": "ClaimsReview",
		"origin": "crm-lead-10040",
		"destination": "claims-work-items@example.biz",
		"createdTime": 1779789108649,
		"eventDetails": "Appended Work Item data successfully.",
		"channelParams": {
			"type": "work-item-form",
			"message": {
				"aliasId": "991d52ea-927f-41ff-832b-d8f8601a8fbd",
				"workItemData": {
					"customerAccountId": "3572653863",
					"customerAccountName": "Updated Customer Name",
					"claimStatus": "Approved"
				},
				"timestamp": 1779789108649,
				"type": "WorkItemForm"
			}
		},
		"reason": null,
		"errorMessage": null
	}
}
```

**Representative `task:failed` subscription webhook**

```json
{
	"id": "8be34eb4-d64f-4552-96cc-c9b6e73063c2",
	"specversion": "1.0",
	"type": "task:failed",
	"source": "/com/cisco/wxcc/ed9357d0-3f5d-4c6a-a4fd-222222222222",
	"comciscoorgid": "cb5901fb-ab66-4377-a7bc-5f0b21896952",
	"comciscotimestamp": 1779785508649,
	"datacontenttype": "application/json",
	"data": {
		"taskId": "141897a0-6212-49be-be9d-4823ab1e4dc7",
		"origin": "crm-lead-10040",
		"destination": "claims-work-items@example.biz",
		"direction": "INBOUND",
		"channel": "ClaimsReview",
		"channelType": "workItem",
		"mediaMgr": "digitalmm",
		"reason": "ENTRY_POINT_NOT_FOUND",
		"errorMessage": "The Work Item asset does not resolve to an active entry point.",
		"failureType": "failed",
		"createdTime": 1779785508649
	}
}

```
**Common failures**
  
  Common cases that can result in `task:failed` include:

- `CHANNEL_ASSET_UNDEFINED`: the destination business address does not resolve to a valid Work Item asset
- `FEATURE_FLAG_DISABLED`: the organization is not enabled for Work Items
- `ENTRY_POINT_NOT_FOUND`: the asset does not resolve to an active entry point
- `ORG_DIGITAL_CONTACT_LIMIT_EXCEEDED`: the organization has reached its active digital contact limit
- `CONVERSATION_CREATION_FAILED`: downstream Work Item creation failed after the request was accepted
- Validation failures, such as missing required schema fields or invalid `workItemData`

**Representative `task-message:append-failed` subscription webhook:**

```json

{
	"id": "64ab4f4a-5c93-4339-b325-3878f289a88f",
	"specversion": "1.0",
	"type": "task-message:append-failed",
	"source": "/com/cisco/wxcc/ed9357d0-3f5d-4c6a-a4fd-333333333333",
	"comciscoorgid": "cb5901fb-ab66-4377-a7bc-5f0b21896952",
	"comciscotimestamp": 1779789108649,
	"datacontenttype": "application/json",
	"data": {
		"taskId": "141897a0-6212-49be-be9d-4823ab1e4dc7",
		"direction": "INBOUND",
		"messageDirection": "INBOUND",
		"channelType": "workItem",
		"channel": "ClaimsReview",
		"origin": "crm-lead-10040",
		"destination": "claims-work-items@example.biz",
		"createdTime": 1779789108649,
		"eventDetails": null,
		"channelParams": null,
		"reason": "INVALID_CONTENT",
		"errorMessage": "Required Work Item schema field claimNumber is missing."
	}
}
```
Use the following fields to correlate Work Item webhook processing

- `data.taskId`: task identifier
- `data.channelParams.message.aliasId`: Developer defined Work Item identifier for `task-message:appended` when `channelParams` is present
- `data.channelParams.message.workItemData`: Work Item metadata for the create or update
- `id`: webhook event identifier

## Security and data handling  

Work Item webhooks use the same signature, timestamp, replay-protection, and version-header validation model as other Webex Contact Center webhooks. For the generic validation steps, see [Using Webhooks](/docs/using-webhooks).

Work Item metadata can contain customer or business-sensitive data. Partners should:  

- Send only fields required for routing, agent handling, reporting, or downstream processing
- Avoid placing secrets in `workItemData`
- Retain only the task identifiers and correlation identifiers needed for the integration
- Treat `workItemData` as customer data in logs, traces, and analytics stores
## Error handling and recovery

Partners should treat `task:failed` and `task-message:append-failed` as the main partner-visible failure events.

**Important guidance:**  

- `task:failed` is the partner-visible failure event for create-task failures
- `task-message:append-failed` is the partner-visible failure event for Work Item update failures
- Representative update-failure reasons include `INVALID_CONTENT`, `INTERNAL_ERROR`, and `NOT_FOUND`

**For request recovery:**

- Validate `destination.id`, `channelType`, `channel`, and `workItemData` before retrying
- Verify the task identifier before retrying Work Item updates
- Correct schema validation failures before retrying
- Retry transient failures with backoff

## Flow Support  

Work Item flows support self-service and routing activities that apply to non-conversational tasks.

Supported Work Item flow activities include:

- `Wait`
- `BRE Request`
- `Condition`
- `Go To`
- `HTTP Request`
- `Case`
- `Parse`
- `End Flow`
- `Percent Allocation`
- `Set Variable`
- `Business Hours`
- `Screen Pop`
- `Set Contact Priority`
- `Escalate CDG`
- `Get Queue Info`
- `Advanced Queue Info`
- `Queue Contact`
- `Queue To Agent`

**Note:** 

- Work Item start-node variables include interaction, entry point, organization, flow, origin, destination, and media payload details. The media payload includes the Work Item metadata supplied in `workItemData`.
- Call handling activities are not shown in Work Item flows. Custom Messaging Send and Receive nodes are not shown in Work Item flows.
## Retrieve Transcripts

Use the [Captures API](/webex-contact-center/docs/api/v1/captures/list-captures) to retrieve post-interaction transcripts for Work Item tasks.

Work Item transcripts contain metadata snapshots rather than conversational messages.

**Representative Work Item transcript entries:**

```json

[
	{
		"id": "51b8f31d-05b2-45f8-b172-63920f2b7d9a",
		"aliasId": "51b8f31d-05b2-45f8-b172-63920f2b7d9a",
		"direction": "inbound",
		"workItemData": {
			"customerAccountId": "3572653863",
			"customerAccountName": "Customer Name",
			"claimNumber": "CLM1234567",
			"priority": "High"
		},
		"timestamp": "2026-06-04T10:24:22.909Z"
	},
	{
		"id": "991d52ea-927f-41ff-832b-d8f8601a8fbd",
		"aliasId": "991d52ea-927f-41ff-832b-d8f8601a8fbd",
		"direction": "inbound",
		"workItemData": {
			"claimStatus": "Approved"
		},
		"timestamp": "2026-06-04T10:28:11.000Z"
	}
]
```
Work Item transcripts are retrieved after transcript generation, subject to retention and capture availability.

  

## Related APIs And Guides

- [Authentication guide](/docs/authentication-cc)
- [Integrations guide](/docs/api/guides/integrations)
- [Create Task API](/webex-contact-center/docs/api/v2/tasks-call-control/create-task)
- [Task Messages API](/webex-contact-center/docs/api/v2/tasks-call-control/append-task-message)
- [End Task API](/webex-contact-center/docs/api/v1/tasks-call-control/end-task)
- [Subscriptions API](/webex-contact-center/docs/api/v2/subscriptions)
- [List Event Types API](/webex-contact-center/docs/api/v2/subscriptions/list-event-types)
- [Webhooks guide](/docs/api/guides/webhooks-cc)
- [Using Webhooks](/docs/using-webhooks)
- [Captures API](/webex-contact-center/docs/api/v1/captures/list-captures)