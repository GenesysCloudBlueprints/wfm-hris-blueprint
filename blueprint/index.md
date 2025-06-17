---
title: Integrate Genesys Cloud with an HRIS system to sync time-off requests and time-off balances from Genesys Cloud WFM
author: Vidas Placiakis
indextype: blueprint
icon: blueprint
image: images/hris_integration_overview.png
category: 12
summary: |
  This Genesys Cloud Developer Blueprint integrates Genesys Cloud with an HRIS to retrieve employee time-off balances and provide that information in Genesys Cloud workforce management. When a time-off request is approved, or an approved request is updated in Genesys Cloud Workforce Management, the integration inserts the request into the HRIS for tracking and verification against the employee's available balance.
---
:::{"alert":"primary","title":"About Genesys Cloud Blueprints","autoCollapse":false} 
Genesys Cloud blueprints were built to help you jump-start building an application or integrating with a third-party partner. 
Blueprints are meant to outline how to build and deploy your solutions, not a production-ready turn-key solution.
 
For more information about Genesys Cloud blueprint support and practices, see our Genesys Cloud blueprint [FAQ](https://developer.genesys.cloud/blueprints/faq "Opens the Blueprint FAQ") sheet.
:::

This Genesys Cloud Developer Blueprint integrates Genesys Cloud with an HRIS to retrieve employee time-off balances and provide that information in Genesys Cloud workforce management. When a time-off request is approved, or an approved request is updated in Genesys Cloud Workforce Management, the integration inserts the request into the HRIS for tracking and verification against the employee's available balance.

You can configure and run this integration from within your Genesys Cloud organization. Architect flows and data actions to sync time-off data with your preferred HRIS. The following image shows a BambooHR integration.

![HRIS integration overview](images/hris_integration_overview.png)

## Solution components

* **Genesys Cloud CX** - A suite of Genesys cloud services for enterprise-grade communications, collaboration, and contact center management. This solution uses Genesys Cloud Architect, integrations and data actions to sync time-off requests and time-off balances with the HRIS.
* **3rd-party HRIS that supports JSON posts to a REST API endpoint** - The integration requires external API access from Genesys Cloud to an HRIS. This blueprint uses BambooHR as an example.

## Prerequisites

### Specialized knowledge

* Experience designing Architect flows
* Experience using the Genesys Cloud Platform API
* Experience with REST API authentication
* Experience with JSON requests
* An understanding of how to use data actions
* Experience with the HRIS you use with this solution, including its time-off features and its API

### HRIS account with API access

Use JSON-based REST APIs to configure an HRIS account in order to provide limited and secure access to its data. 

Genesys Cloud workforce management integration needs sufficient API access to complete the following tasks:
  * Get a list of configured time-off types
  * Get a list of agents whose time-off data is synced by the integration
  * Get an HRIS agent-unique key or ID 
  * Map the email address of a user in the HRIS to the corresponding agent in Genesys Cloud
  * Retrieve information about agents' time-off balance on requested dates, for specific types of time-off 
  * Insert an agent's time-off request 
  * Modify and delete previously inserted agent time-off requests 

### Genesys Cloud account requirements

* One of the following Genesys Cloud licenses. For more information, see [Genesys Cloud Pricing](https://www.genesys.com/pricing "Opens the Genesys Cloud Pricing article") in the Genesys website.
  * Genesys Cloud CX 3
  * Genesys Cloud CX 3 Digital
  * Genesys Cloud CX 1 WEM Upgrade 2
  * Genesys Cloud CX 2 WEM Upgrade 1
  * Genesys Cloud EX
* The Master Admin role in Genesys Cloud. For more information, see [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Opens the Roles and permissions overview article") in the Genesys Cloud Resource Center.

For more information, see [Roles and permissions overview](https://help.mypurecloud.com/articles/about-roles-permissions/ "Opens the Roles and permissions overview article") in the Genesys Cloud Resource Center.

## Solution architecture

This solution is based on Architect flows, which are activated by Genesys Cloud Workforce Management application. The flows use data actions to get or modify HRIS data. You can also build additional logic into the flows to manipulate HRIS data however you like.

Credentials stored in the integration's configuration are used to authorize data actions to access the specified route of the HRIS JSON REST API endpoint.

### Example Architect flows

The integration provides the following examples of Architect flows:

* [HRIS-Get-Agents flow](#hris-get-agents-flow "Goes to the HRIS-Get-Agents flow section")
* [HRIS-Get-Timeoff-Types flow](#hris-get-timeoff-types-flow "Goes to the HRIS-Get-Timeoff-Types section")
* [HRIS-Get-Balance flow](#hris-get-balance-flow "Goes to the HRIS-Get-Balance flow section")
* [HRIS-Insert-TimeOff flow](#hris-insert-timeoff-flow "Goes to the HRIS-Insert-TimeOff flow section")
* [HRIS-Update-TimeOff flow](#hris-update-timeoff-flow "Goes to the HRIS-Update-TimeOff flow section")

:::primary
**Note** These flows can be updated as needed for your specific business purposes.
:::

## HRIS-Get-Agents Workflow

This workflow retrieves unique agent IDs and email addresses from an external HRIS system. The email address in the HRIS system must exactly match the email address defined in the corresponding Genesys Cloud user record. Using the email address, Genesys Cloud matches each user record with its associated external HRIS ID. Genesys Cloud then uses this external ID as an input parameter when invoking workflows to retrieve agent time-off balances or to create or update time-off requests.

The workflow is expected to retrieve all agents associated with the integration each time it runs. If an agent previously associated with an integration is not returned in a subsequent run, Genesys Cloud will remove the association and clear the external ID for that agent.

This workflow is optional. External agent IDs can also be set manually using the Genesys Cloud UI or API.

### Invocation

If automatic synchronization is enabled in configuration, the workflow runs every 24 hours to synchronize agents newly added in the external HRIS system with Genesys Cloud.

The workflow runs for the first time shortly after the initial HRIS integration is activated for an organization. From that point forward, all “Get Agents” workflows across integrations for the organization run on the same 24-hour schedule, with their start time based on the first activation timestamp.

Manual triggering or scheduling of synchronization is not supported.

### Input

Genesys Cloud does not pass any input parameters to this workflow.

### Output

The workflow outputs two string arrays — emails and externalIds — each with a maximum of 2,000 entries. To support up to 50,000 agents, the output includes 25 buckets for each array (emails through emails24, and externalIds through externalIds24).

Each pair of arrays in the same bucket must contain the same number of entries, and their order must align to ensure correct matching of email addresses with external IDs.

| Name               |           Data Type           | Notes                                                                    | Mandatory |
|:-------------------|:-----------------------------:|:-------------------------------------------------------------------------|:----------|
| Flow.statusCode    |  Integer (HTTP status code)   | 200 on success, 500 on error, 408 on timeout                             | Yes       |
| Flow.status        |            String             | Set to “Complete” on success; otherwise, set to “Error”                  | Yes       |
| Flow.errorMsg      |            String             | Error message if the workflow fails                                      | No        |
| Flow.emails        |         String Array          | Email addresses from the external HRIS system that match Genesys users   | Yes       |
| Flow.externalIds   |         String Array          | External agent IDs corresponding to the matched email addresses          | Yes       |
| Flow.emails1       |         String Array          | Additional bucket for email addresses                                    | No        |
| Flow.externalIds1  |         String Array          | Additional bucket for external IDs                                       | No        |
| ..                 |              ..               | ..                                                                       |           |
| Flow.emails24      |         String Array          | Additional buckets (up to 24) for email addresses                        | No        |
| Flow.externalIds24 |         String Array          | Additional bucket for external IDs                                       | No        |

## HRIS-Get-Timeoff-Types Workflow

This workflow retrieves a list of time-off types from an external HRIS system. Each time-off type includes a primary ID (or key), a name, and an optional secondary ID. Genesys Cloud WFM uses this information when checking an agent’s time-off balance or when creating or updating time-off requests.

The optional secondary ID can be used when a single ID is insufficient to identify a time-off type and its corresponding accrual rule in the HRIS. Usage of the secondary ID depends on the specific HRIS. When present, the secondary ID is included in subsequent requests from Genesys Cloud WFM to the HRIS.

### Invocation

This workflow is invoked through the Genesys Cloud WFM admin interface when a time-off plan is associated with an external HRIS integration. Upon selecting the HRIS integration, the system queries available time-off types from the external HRIS. The administrator can then map an external time-off type to the corresponding Genesys time-off plan and activity codes.

### Input

Genesys Cloud does not pass any input parameters to this workflow.

### Output

The workflow returns three string arrays representing the IDs, names, and optional secondary IDs of time-off types from the external HRIS system. The number of entries in all returned arrays must match, and the order must align to ensure correct mapping. 

| Name               |           Data Type           | Notes                                                                | Mandatory |
|:-------------------|:-----------------------------:|:---------------------------------------------------------------------|:----------|
| Flow.statusCode    |  Integer (HTTP status code)   | 200 on success, 500 on error, 408 on timeout                         | Yes       |
| Flow.status        |            String             | Set to “Complete” on success; otherwise, set to “Error”              | Yes       |
| Flow.errorMsg      |            String             | Error message if the workflow fails                                  | No        |
| Flow.ids           |         String Array          | Primary IDs of time-off types configured in the external HRIS system | Yes       |
| Flow.names         |         String Array          | Names of time-off types                                              | Yes       |
| Flow.id2s          |         String Array          | Optional secondary IDs for time-off types                            | No        |

## HRIS-Get-Balance Workflow

This workflow retrieves time-off balances for a given agent, based on requested dates and external time-off types. Genesys Cloud WFM uses this workflow to present time-off balances for the requested periods. It does **not** verify whether an agent is permitted to take additional time off.

### Invocation

This workflow is invoked via the Genesys Cloud WEM admin interface and agent time-off request screens whenever administrators or agents view time-off balances.

### Input

If provided, the array of secondary time-off type IDs will have the same number of entries and follow the same order as the primary time-off type IDs.

| Name                    |   Data Type  | Notes                                                                   | Mandatory |
|:------------------------|:------------:|:------------------------------------------------------------------------|:----------|
|Flow.inputDates          | String Array | Dates for which time off balance is requested (in YYYY-MM-DD format)    | Yes       |
|Flow.inputTimeOffTypeIds | String Array | External time-off type IDs to check balances for                        | Yes       |
|Flow.agentId             |    String    | External HRIS agent ID                                                  | Yes       |
|Flow.inputTimeOffTypeId2s| String Array | Optional secondary time-off type IDs, if defined for the time-off types | No        |

### Output

All output arrays must be the same length. This length equals the product of the number of input dates and the number of time-off types. There should be one entry for every combination of date and time-off type. The order of items across all output arrays must be aligned, although the order itself does not matter.

If a secondary time-off type ID was provided in the input, it must be included alongside the corresponding primary ID in the output.

| Name                     | Data Type                  | Notes                                                                                    | Mandatory |
|:-------------------------|:---------------------------|:-----------------------------------------------------------------------------------------|:----------|
| Flow.statusCode          | Integer (HTTP status code) | 200 on success, 500 on error, 408 on timeout                                             | Yes       |
| Flow.status              | String                     | Set to “Complete” on success; otherwise, set to “Error”                                  | Yes       |
| Flow.errorMsg            | String                     | Error message if the workflow fails                                                      | No        |
| Flow.balanceMinutesPerDay| String Array               | Time-off balances in minutes for the specified dates and time-off types                  | Yes       |
| Flow.timeOffTypeIds      | String Array               | External time-off type IDs for which balances were returned                              | Yes       |
| Flow.dates               | String Array               | Dates (in YYYY-MM-DD format) for which balances were returned                            | Yes       |
| Flow.timeOffTypeId2s     | String Array               | Optional secondary external time-off type IDs                                            | No        |

## HRIS-Insert-TimeOff Workflow

This workflow inserts a new time-off request into the external HRIS. If the insertion is successful, the workflow returns the external time-off request ID generated by the HRIS.

If the request is submitted in a `PENDING` status, the workflow should first verify the agent’s time-off balance. If the agent’s balance is insufficient, it must set the `status` to `InsufficientBalance`. This response prevents the time-off request from being auto-approved in Genesys Cloud WFM.

If the workflow invocation returns an error, the time-off synchronization status in Genesys Cloud WFM is set to "failed"".

### Invocation

This workflow is invoked when a time-off request is submitted via the Genesys Cloud WFM admin interface (for approved requests) or via either the admin or agent interface (for pending requests). To trigger the workflow, the time-off request must be linked to a time-off plan in Genesys Cloud WFM, and that plan must be associated with an HRIS integration.

### Input

One of the `dates` or `startDateTimes` arrays will be provided to the flow as input. This array is aligned with the `payableMinutes` array.

| Name                  | Data Type        | Notes                                                                                                      | Mandatory |
|-----------------------|------------------|------------------------------------------------------------------------------------------------------------|-----------|
| Flow.agentId          | String           | External HRIS agent ID for whom the time-off request is being inserted                                     | Yes       |
| Flow.dates            | String Array     | Dates for the time-off request in `YYYY-MM-DD` format; includes all days spanning the request              | Yes       |
| Flow.startDateTimes   | String Array     | For partial-day time-off requests only; ISO datetime strings in `YYYY-MM-DDTHH:mm:ss` format               | No        |
| Flow.minDate          | String           | Earliest date of the request (in `YYYY-MM-DD` format)                                                      | Yes       |
| Flow.maxDate          | String           | Latest date of the request (in `YYYY-MM-DD` format)                                                        | Yes       |
| Flow.timeOffStatus    | String           | One of: `APPROVED`, `PENDING`                                                                              | Yes       |
| Flow.timeOffTypeId    | String           | External time-off type ID configured in the HRIS                                                           | Yes       |
| Flow.payableMinutes   | String Array     | Payable minutes for each date in the request                                                               | Yes       |
| Flow.notes            | String           | Optional notes attached to the request                                                                     | No        |

### Output

The workflow should return an external `timeOffRequestId` if it successfully adds the time-off request to the external HRIS. If the agent does not have enough balance, the workflow must return `statusCode` as `200`, with the `status` set to `InsufficientBalance`.

| Name                  | Data Type                  | Notes                                                                                                   | Mandatory |
|-----------------------|----------------------------|---------------------------------------------------------------------------------------------------------|-----------|
| Flow.statusCode       | Integer (HTTP status code) | 200 on success, 500 on error, 408 on timeout                                                            | Yes       |
| Flow.status           | String                     | `Complete` on success; `InsufficientBalance` if the agent lacks sufficient balance; otherwise, `Error`  | Yes       |
| Flow.errorMsg         | String                     | Error message if the workflow fails                                                                     | No        |
| Flow.timeOffRequestId | String                     | External `timeOffRequestId` created in the HRIS                                                         | No        |

## HRIS-Update-TimeOff Workflow

This workflow updates a time-off request in an external HRIS system after it has been modified in Genesys Cloud WFM. If the `timeOffStatus` input is `CANCELED` or `DENIED`, the workflow may remove or similarly cancel the time-off request in the external HRIS.

If the request is submitted in a `PENDING` status, the workflow should first verify the agent’s time-off balance. If the agent’s balance is insufficient, it must set the `status` to `InsufficientBalance`. This response prevents the time-off request from being auto-approved in Genesys Cloud WFM.


Genesys Cloud WFM marks the time-off request as successfully synchronized. If the workflow invocation fails, the synchronization status is set to FAILED.

### Invocation

This flow is invoked, when time-off request that has been previously inserted in external HRIS is changing within Genesys Cloud WFM. 

### Input

One of the `dates` or `startDateTimes` arrays will be provided to the flow as input. This array is aligned with the `payableMinutes` array.

| Name                  | Data Type        | Notes                                                                                                      | Mandatory |
|-----------------------|------------------|------------------------------------------------------------------------------------------------------------|-----------|
| Flow.agentId          | String           | External HRIS agent ID for whom the time-off request is being updated                                      | Yes       |
| Flow.dates            | String Array     | Dates for the time-off request in `YYYY-MM-DD` format; includes all days spanning the request              | Yes       |
| Flow.startDateTimes   | String Array     | For partial-day time-off requests only; ISO datetime strings in `YYYY-MM-DDTHH:mm:ss` format               | No        |
| Flow.minDate          | String           | Earliest date of the request (in `YYYY-MM-DD` format)                                                      | Yes       |
| Flow.maxDate          | String           | Latest date of the request (in `YYYY-MM-DD` format)                                                        | Yes       |
| Flow.timeOffStatus    | String           | One of: `APPROVED`, `PENDING`, `DENIED`, `CANCELED`                                                        | Yes       |
| Flow.timeOffTypeId    | String           | External time-off type ID configured in the HRIS                                                           | Yes       |
| Flow.payableMinutes   | String Array     | Payable minutes for each date in the request                                                               | Yes       |
| Flow.notes            | String           | Optional notes attached to the request                                                                     | No        |

### Output

The workflow should return the external `timeOffRequestId` if it successfully updates the request in the HRIS. If the agent does not have ensufficient balance, it must return `statusCode` as `200`, with the `status` set to `InsufficientBalance`.

| Name                   | Data Type                  | Notes                                                                                                   | Mandatory |
|------------------------|----------------------------|---------------------------------------------------------------------------------------------------------|-----------|
| Flow.statusCode        | Integer (HTTP status code) | 200 on success, 500 on error, 408 on timeout                                                            | Yes       |
| Flow.status            | String                     | `Complete` on success; `InsufficientBalance` if balance is insufficient; otherwise, `Error`             | Yes       |
| Flow.errorMsg          | String                     | Error message if the workflow fails                                                                     | No        |
| Flow.timeOffRequestId  | String                     | External `timeOffRequestId` associated in the HRIS                                                      | Yes       |


### Example data actions

Integration provides data actions in the following examples:

* **Bamboo-Get-TimeOff-Types** - The HRIS provides a list of time off types in this data action.
* **Bamboo-Get-Employees** - Obtains the employee list from the HRIS.
* **Bamboo-Get-Balance** - Estimates the time-off balance of an employee in the HRIS. The example uses the "v1/employees/{employeeId}/time_off/calculator" route of BambooHR HRIS, which returns a rounded balance to one decimal point.
* **Bamboo-Put-TimeOff-Request** - Inserts a time-off request from Genesys Cloud into the HRIS. Both the ### HRIS-Insert-TimeOff flow and the HRIS-Update-TimeOff flow use this data action.

:::primary
**Note** Once imported into Genesys Cloud, these data actions can be updated to meet your specific business needs.
:::

## Implementation steps

### Clone the GitHub repository

:::primary
**Note**: The GitHub repository should only be cloned if you plan to use the example flows and data actions that integrate with BambooHR HRIS. Otherwise, modify the example files or skip this step. You can create data actions and flows directly in your Genesys Cloud organization.
:::

Clone the [wfm-hris-blueprint](https://github.com/GenesysCloudBlueprints/wfm-hris-blueprint) repository to your local machine. The examples folder contains the Architect flows that can be modified to meet your needs.

### Create the integration for data actions

Add a web services data actions integration to Genesys Cloud to enable communication with your HRIS:

1. In Genesys Cloud, navigate to **Admin** > **Integrations** and install a **Web Services Data Actions** integration from Genesys Cloud. For more information, see [About the data actions integrations](https://help.mypurecloud.com/?p=209478 "Opens the data actions overview article") in the Genesys Cloud Resource Center.

  ![Web services data actions integration tile](images/integrations_install.png "Web services data actions integration tile")

2. Rename the web services data action integration and provide a short description.
3. Click **Configuration** > **Credentials** and then click **Configure**.

   ![Configure integration credentials](images/data_actions_credentials.png "Click Configure")

4. From the **Credential Type** list, select **User Defined (OAuth)** and configure the credentials to access your HRIS.
5. Click **Save**.

### Create the data actions

This solution provides [example data actions](#example-data-actions "Goes to the Example data actions section"). Data actions are performed using JSON API calls to the HRIS. A specific API route is configured for each data action, which exchanges specific HRIS data.

Follow the same instructions for each of these data actions or the newly created data actions.

:::primary
**Note**
These instructions demonstrate how to use the example data actions in the blueprint solution. The steps you need to follow depend on your HRIS.
:::

1. In Genesys Cloud, navigate to **Admin** > **Integrations** > **Actions**.
2. Follow these steps:
  * Click **Import** to use an example data action.
  * Click **Add** to create a new data action.

  ![Create Action](images/data_actions_create.png)
3. Select the data action file from your local copy of the repository.
4. For the Integration name, use the name of the [web services data actions integration that you created in the previous step](#create-the-integration "Goes to the Create the integration section").
5. Specify a name in the **Action Name** field.
6. Click **Setup** and configure the **Contracts** and **Configuration** sections. It is important to note that the **Contracts** section has input and output contracts that your flows use.
7. Click **Publish Data Action**.

For more information, see [About the web services data actions integration](https://help.mypurecloud.com/?p=127163 "Goes to the About the web services data actions integration article") in the Genesys Cloud Resource Center.

### Import the Architect flows

This solution provides [example Architect flows](#Example-architect-flows "Goes to the Example Architect flows section"), which you need to import into your Genesys Cloud organization.

:::primary
**Note**: The example flow can be modified once it has been added and imported.
:::

1. In Genesys Cloud, navigate to **Admin** > **Architect**.
2. From the **Flows** drop-down list, select **Workflow**.
![Select a flow](images/architect_select_workflow.png "Select a flow")
3. Click **Add**.
4. Enter a name and description for the example flow you are importing.
![Add a flow](images/architect_add_workflow.png "Add a flow")
5. Click **Create**.
6. Click **Import** and select the example workflow you are importing.
![Import a flow](images/architect_import_workflow.png "Import a flow")
7. Make any necessary changes.
8. Click **Save** and **Publish**.

For more information, see [About Architect](https://help.mypurecloud.com/?p=53682 "Goes to the About Architect article") and [Work with workflows](https://help.mypurecloud.com/?p=215071 "Goes to the Work with workflows article") in the Genesys Cloud Resource Center.

### Create the WFM Time-off HRIS integration

1. In Genesys Cloud, navigate to **Admin** > **Integrations** and install a **WFM Time-off HRIS** integration from Genesys Cloud. 
2. Rename the integration and provide a short description.
3. Click **Configuration** > **Properties**.
4. Select previously imported or created workflows for every listed task. 
5. There should be valid workflows selected for all configuration properties except the agent sync workflow which is optional. Attached is the screenshot with the selection

![img_2.png](images/wfm_hris_configuration.png)

6. Click **Save**.

## Test the workflows with the Genesys Cloud API

### Select test tool

There are two convenient ways to make API calls to Genesys Cloud: Postman and [Genesys Cloud API Explorer](https://developer.genesys.cloud/devapps/api-explorer). Below you'll find an information about both tools usage.

As a client, you can also use your preferred method to call an API, for instance, curl.

#### Use Postman to test workflows

Postman is a popular third-party tool to make REST API requests. Genesys Cloud provides Postman collection among with environment files to simplify. Please, refer to [Genesys Cloud Developer Center - Postman](https://developer.genesys.cloud/platform/api/postman) for detailed information and authentication instructions.


#### Use Genesys Cloud API Explorer to test workflows

[API Explorer](https://developer.genesys.cloud/devapps/api-explorer) provides convenient way to test your workflows with built-in authentication. For detailed usage guide, please refer to [API Explorer usage](https://developer.genesys.cloud/devapps/about/api-explorer).

Please, refer to [API Explorer Documentation](https://developer.genesys.cloud/devapps/about/api-explorer) for detailed usage instructions. 

* Under Category list in right area of the page, click "Clear all" and select Architect - you'll see list of API endpoints
* Locate POST to "/api/v2/flows/executions", click it - this is endpoint to trigger workflow execution
* Disable "Reading mode" toggle
* Enable "Pro Mode" toggle to switch to raw json input. Click "Load empty schema" and fill it with your values.
![alt text](images/api-explorer.png)
* Add values to invoke execution of different workflows
* You can find examples of "inputData" and other values below in this document, in [Examples of requests](#examples-of-requests) section.


### Authentication

Skip this section if you're using [API Explorer](https://developer.genesys.cloud/devapps/api-explorer).
For Postman usage, please, refer to [Genesys Cloud Developer Center - Postman](https://developer.genesys.cloud/platform/api/postman)

If you're using some other third-party client, such as curl, authenticate your client and get a bearer token to call an API. For more information, please refer to [Grant - Authorization Code](https://developer.genesys.cloud/authorization/platform-auth/use-authorization-code).

### Obtain workflow id

To be able to test, for each imported workflow you'll need its unique ID. 

To da that, sens GET to /api/v2/flows with workflow name as a query parameter using your selected test tool. For example: /api/v2/flows?name="my_flow_name".

Alternatively: open published Workflow in Architect and locate Id in your browser address line as shown below:
![alt text](images/architect_workflow_id.png)


### Trigger workflow execution

Every workflow execution could be triggered by [POST /api/v2/flows/executions](https://developer.genesys.cloud/routing/architect/#post-api-v2-flows-executions "Opens the POST /api/v2/flows/executions"). Below you can find examples of correct requests bodies for each workflow.

As a result of each request, you will receive an individual {flowExecutionId}, which will be used later.

### Get flow execution result

Once {flowExecutionId} is received, you can check execution result by sending [GET /api/v2/flows/executions/{flowExecutionId}](https://developer.genesys.cloud/routing/architect/#get-api-v2-flows-executions--flowExecutionId- "Opens the GET /api/v2/flows/executions/{flowExecutionId}"). This request doesn't require any data or parameters.

### Examples of requests

Below you can find examples of requests and expected results for each sample workflow.


#### HRIS-Get-Agents-Flow

```
POST /api/v2/flows/executions
{
  "flowId": "${flow_id_get_agents}"
}
```

#### HRIS-Get-Timeoff-Types flow

```
POST /api/v2/flows/executions
{
  "flowId": "${flow_id_get_timeoff_types}"
}
```

#### HRIS-Get-Balance flow

```
POST /api/v2/flows/executions
{
  "flowId": "${flow_id_get_balance}",
  "inputData": {
    "Flow.inputDates": ["2025-06-01"],
    "Flow.inputTimeOffTypeIds":[
            "83",
            "90",
            "85"
        ],

    "Flow.agentId": "452"
    
  }
}
```

#### HRIS-Insert-TimeOff flow
```
POST /api/v2/flows/executions
{
  "flowId": "${flow_id_insert_timeoff}",
  "inputData": {
    "Flow.agentId": "452",
    "Flow.notes": "Testing timeoff",
    "Flow.dates":["2025-03-29", "2025-03-30", "2025-03-31"],
    "Flow.minDate": "2025-03-29",
    "Flow.maxDate" : "2025-03-31",
    "Flow.timeOffStatus":"APPROVED",
    "Flow.timeOffTypeId":"83",
    "Flow.payableMinutes":["480", "480", "480"]
  }
}
```

#### HRIS-Update-TimeOff flow
```
POST /api/v2/flows/executions
{
  "flowId": "${flow_id_update_timeoff}",
  "inputData": {
    "Flow.agentId": "452",
    "Flow.notes": "Testing timeoff",
    "Flow.dates":["2025-03-29", "2025-03-30", "2025-03-31", "2025-04-01"],
    "Flow.minDate": "2025-03-29",
    "Flow.maxDate" : "2025-04-01",
    "Flow.timeOffStatus":"APPROVED",
    "Flow.timeOffTypeId":"83",
    "Flow.payableMinutes":["480", "480", "480", "480"],
    "Flow.inputTimeOffRequestId":"{{previous_timeoff_id}}"
  }
}
```
## Additional resources

* [wfm-hris-blueprint repository](https://github.com/GenesysCloudBlueprints/wfm-hris-blueprint "Opens the wfm-hris-blueprint repository") in GitHub.
