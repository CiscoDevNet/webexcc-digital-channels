---
title: Work Items
display_title: Work Items
url: /docs/work-items
navigation_category:
- webex-contact-center
product_group:
- webex-contact-center
search_category: Documentation
---

Work Items enable external systems to inject non-conversational tasks into Webex Contact Center so that those tasks can be routed, handled, monitored, and reported alongside other contact center interactions.

Work Items do not represent live messaging conversations. Use Work Items for backend or offline tasks such as CRM leads, claims, faxes, case reviews, fulfillment tasks, or other work that needs contact center routing and agent handling without a customer chat thread.

Availability for Work Items is controlled by the `wxcc_digital_work_item` feature toggle. Contact your Cisco team to request access.

In this model:

- the external system owns the source task and work item metadata
- Webex Contact Center owns routing, flows, agent handling, task state, lifecycle events, and reporting
- administrators configure the Work Item channel, asset, business address, schema, entry point, flow, queues, and capacity limits
- developers use public APIs to create Work Item tasks, update Work Item data, subscribe to lifecycle events, and retrieve Work Item transcripts

## Responsibilities

Use this model when your application is responsible for creating or updating non-conversational business tasks in Webex Contact Center.

The partner application is responsible for:

- hosting the system that receives or produces the external work
- obtaining and refreshing OAuth tokens
- calling the Create Task API for the initial Work Item
- calling the Task Messages API when Work Item metadata must be updated
- providing `workItemData` that conforms to the administrator-configured Work Item schema
- optionally subscribing to task lifecycle and Work Item update webhooks
- retrieving Work Item transcripts when post-interaction metadata is needed

Webex Contact Center is responsible for:

- validating Work Item task requests against configured assets, entry points, and schema
- running the configured Work Item flow
- routing Work Items to queues and agents
- presenting Work Item metadata to agents as read-only task data
- generating task lifecycle and Work Item update webhook events
- storing Work Item metadata snapshots for later retrieval

## Before You Begin

Before integrating, ensure that the administrator has configured the following in Control Hub:

- a `Work Item` channel
- a `Work Item` asset with:
  - asset name and description
  - business address
  - data schema fields
- a `Work Item` entry point mapped to a `Work Item` flow
- Work Item queues and queue rankings, as needed
- multimedia profile capacity for Work Items
- RONA timeout values for Work Items
- other Contact Center routing objects required for the interaction, such as teams, skills, queues, and agent profiles

Work Item assets use a configured business address to resolve Create Task requests. The business address must be unique across Work Item assets.

Work Item schema configuration defines which metadata fields are accepted for the task. A schema field includes:

- field key
- display name
- whether the field is required
- whether the field is viewable

The configured schema can contain up to 50 fields. Each field key can contain up to 100 characters, and each field value can contain up to 1024 characters. At least one required field and one viewable field must be configured before a Work Item can be saved.

## Authentication For Work Item Integrations

The general [Authentication guide](/docs/authentication-cc) explains token acquisition and token refresh behavior. For Work Item partner integrations, you should also complete the following setup so the token can be used for the Tasks APIs.

If you are building a long-running machine-to-machine integration, use a [Service App](/docs/service-apps) so that token handling and customer authorization are managed appropriately for server-side operation.

For scope definitions, refer to the [Integrations guide](/docs/api/guides/integrations).

### Create And Authorize A Service App

1. Sign in to the developer portal and create a new Service App.
1. Capture the client secret when it is first shown. After the app is submitted for authorization, regenerate the secret if it was not saved.
1. Assign the required scopes:
   - `cjp:task_write`
   - `cjp:task_read`
1. Ask the customer administrator to authorize the Service App in Control Hub under **Apps**.
1. After authorization, return to the Service App in the developer portal, select the authorized organization, provide the client secret, and generate the access token.

Use the resulting access token when calling the Create Task API and the Task Messages API.

For this integration pattern, ensure that:

- the Service App is authorized for the customer organization
- the access token includes `cjp:task_write` for Create Task and Task Messages
- the access token includes `cjp:task_read` when task read access is required

### Refresh Tokens

If your integration needs long-running access, retain the refresh token and use the standard OAuth refresh flow described in the [Authentication guide](/docs/authentication-cc) before the current access token expires.

For server-to-server integrations, store the client secret, access token, and refresh token securely, and rotate or regenerate credentials according to your organization's security practices.

## Work Item Schema Behavior

Work Item schema is configured on the Work Item asset. The schema controls the metadata fields that can be supplied when creating or updating a Work Item.

For Work Item API requests:

- `workItemData` must contain at least one field
- field keys must match keys configured in the Work Item asset schema
- required schema fields must be present and non-empty
- required fields sent with `null` are treated as empty
- values are validated against configured schema constraints
- invalid, missing, unknown, or empty required fields can cause the request to fail

Work Items do not support attachments. Do not include Custom Messaging attachment payloads, message text policies, or conversational Send/Receive behavior in Work Item requests.

## Create The Initial Work Item

Use the [Create Task API](/webex-contact-center/docs/api/v2/tasks-call-control/create-task) to create the initial Work Item task.

Example request:

```json
{
  "origin": {
    "id": "crm-lead-10040",
    "name": "Customer Name"
  },
  "destination": {
    "id": "claims-work-items@example.biz",
    "type": "businessAddress"
  },
  "channelType": "workItem",
  "channel": "ClaimsReview",
  "flowSettings": {
    "sourceSystem": "crm"
  },
  "globalVariables": {
    "customerTier": "gold"
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

Important request fields:

- `origin.id`: partner-managed customer, case, task, or source-system identifier
- `origin.name`: display name associated with the Work Item origin
- `destination.id`: business address configured on the Work Item asset
- `destination.type`: must be `businessAddress`
- `channelType`: must be `workItem`
- `channel`: configured Work Item channel name
- `flowSettings`: optional flow settings supplied to the task
- `globalVariables`: optional Contact Center global variables used by flow and routing logic
- `channelParams.type`: must be `work-item-form`
- `channelParams.message.aliasId`: partner-provided Work Item data identifier for correlation
- `channelParams.message.workItemData`: metadata fields that conform to the Work Item asset schema
- `channelParams.message.timestamp`: Work Item timestamp in epoch milliseconds

The guide shows one representative Work Item example. For the full set of supported request combinations and validation rules, refer to the [Create Task API](/webex-contact-center/docs/api/v2/tasks-call-control/create-task).

Example response:

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

Use `data.id` as the created task identifier. Partners should retain this value for subsequent Task Messages API calls and to correlate later task lifecycle subscription webhooks for the interaction.

## Update Work Item Data

Use the [Task Messages API](/webex-contact-center/docs/api/v2/tasks-call-control/append-task-message) to append updated Work Item metadata to the same task.

For Work Items, this API appends a metadata update. It does not append a conversational chat message.

Example request:

```json
{
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
  }
}
```

Example response:

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

Use `data.id` as the update event correlation identifier. If request `aliasId` is a valid UUID, the platform can reuse that value as the event identifier.

After a successful update, Webex Contact Center publishes the inbound `task-message:appended` subscription webhook for the same Work Item update, and the detailed event payload is documented in [Webhook Event Details](/docs/api/guides/webhooks-cc).

Important request fields:

- `channelParams.type`: must be `work-item-form`
- `channelParams.message.aliasId`: partner-provided Work Item update identifier for correlation
- `channelParams.message.workItemData`: metadata fields to append to the Work Item transcript
- `channelParams.message.timestamp`: Work Item update timestamp in epoch milliseconds
- `channelParams.message.type`: must be `WorkItemForm`

Common Work Item update failure categories surfaced through `task-message:append-failed` are shown below, and the detailed event payload is documented in [Webhook Event Details](/docs/api/guides/webhooks-cc):

| Failure reason | Typical meaning | Typical next step |
| --- | --- | --- |
| `INVALID_CONTENT` | The request payload violates Work Item schema or validation rules, such as missing required fields or invalid `workItemData`. | Correct the request payload and retry. |
| `NOT_FOUND` | The referenced task context could not be found. | Verify the task identifier before retrying. |
| `INTERNAL_ERROR` | The platform could not complete the update because of an unexpected internal failure. | Investigate the related failure event and retry. |

## Subscribe To Task Lifecycle Events

Use the [Subscriptions API](/webex-contact-center/docs/api/v2/subscriptions) for task lifecycle and Work Item update webhook events.

Important event families for Work Item integrations include:

- `task:new`
- `task:connect`
- `task:connected`
- `task:ended`
- `task:failed`
- `task-message:appended`
- `task-message:append-failed`

Use the [List Event Types API](/webex-contact-center/docs/api/v2/subscriptions/list-event-types) to discover the latest supported event types and resource versions.

Use the [Webhooks guide](/docs/api/guides/webhooks-cc) for detailed task and task-message event payload documentation.

For Work Items, subscription-based `task-message:appended` and `task-message:append-failed` describe Work Item metadata updates. They do not represent customer chat messages.

Representative Work Item `task-message:appended` subscription webhook:

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

Representative `task:failed` subscription webhook:

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

Common cases that can result in `task:failed` include:

- `CHANNEL_ASSET_UNDEFINED`: the destination business address does not resolve to a valid Work Item asset
- `FEATURE_FLAG_DISABLED`: the organization is not enabled for Work Items
- `ENTRY_POINT_NOT_FOUND`: the asset does not resolve to an active entry point
- `ORG_DIGITAL_CONTACT_LIMIT_EXCEEDED`: the organization has reached its active digital contact limit
- `CONVERSATION_CREATION_FAILED`: downstream Work Item creation failed after the request was accepted
- validation failures, such as missing required schema fields or invalid `workItemData`

Representative `task-message:append-failed` subscription webhook:

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

Use the following fields to correlate Work Item webhook processing:

- `data.taskId`: task identifier
- `data.channelParams.message.aliasId`: partner Work Item identifier for `task-message:appended` when `channelParams` is present
- `data.channelParams.message.workItemData`: Work Item metadata for the create or update
- `id`: webhook event identifier

## Security And Data Handling

Work Item webhooks use the same signature, timestamp, replay-protection, and version-header validation model as other Webex Contact Center webhooks. For the generic validation steps, see [Using Webhooks](/docs/using-webhooks).

Work Item metadata can contain customer or business-sensitive data. Partners should:

- send only fields required for routing, agent handling, reporting, or downstream processing
- avoid placing secrets in `workItemData`
- retain only the task identifiers and correlation identifiers needed for the integration
- treat `workItemData` as customer data in logs, traces, and analytics stores

## Error Handling And Recovery

Partners should treat `task:failed` and `task-message:append-failed` as the main partner-visible failure events.

Important guidance:

- `task:failed` is the partner-visible failure event for create-task failures
- `task-message:append-failed` is the partner-visible failure event for Work Item update failures
- representative update-failure reasons include `INVALID_CONTENT`, `INTERNAL_ERROR`, and `NOT_FOUND`

For request recovery:

- validate `destination.id`, `channelType`, `channel`, and `workItemData` before retrying
- verify the task identifier before retrying Work Item updates
- correct schema validation failures before retrying
- retry transient failures with backoff

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

Call handling activities are not shown in Work Item flows. Custom Messaging Send and Receive nodes are not shown in Work Item flows.

Work Item start-node variables include interaction, entry point, organization, flow, origin, destination, and media payload details. The media payload includes the Work Item metadata supplied in `workItemData`.

## Retrieve Transcripts

Use the [Captures API](/webex-contact-center/docs/api/v1/captures/list-captures) to retrieve post-interaction transcripts for Work Item tasks.

Work Item transcripts contain metadata snapshots rather than conversational messages.

Representative Work Item transcript entries:

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
