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

## <u>HRIS-Get-Agents flow</u>

The purpose of this workflow is to to provide unique ids of agents in external HRIS system together with their emails, according to which these agents can be mapped to the user records in Genesys Cloud. 

This workflow is expected to return a list of emailIds and associated externalIds of the agents from external HRIS system that is to be synchronized with Genesys users. 

The number of records in both lists should match. The list order is important as the system will use the order to match elements in list of ids with the list of emails.

The email ID of a given agent configured in external HRIS system should match to that of the user defined in Genesys and that is the key based on which the agents in two systems are integrated.

The flow does not have need a input parameter and the out of the workflow would be a list of emailId and externalId both of which are string data types. Below is a table showing that information.

Note that a workflow variable can only have up to 2000 entries maximum and hence to facilitate that there are 25 buckets of emails and externalIds provided to send up 50K agent details.
The indexed of additional buckets start from 1 through 24 in addition to the initial bucket. 

The workflow is assigned to the WFM Integration configuration property of "User Account IDs" that has a description of "An architect workflow to retrieve a list of users from HRIS". This will ensure the workflow
is triggered as part of scheduled agent synchronization process.

### <u>Flow invocation context and details</u>

The flow is invoked through a scheduler on daily basis (every 24 hours) to ensure any new agents added to external HRIS system is picked up and synced with Genesys. The linkage happens via email configured for a user
defined in external system. It is imperative that the email used in user definition matches to that of user defnition in Genesys for synchronization to successfully happen. The synchronization begins for an organization
first time once the very first HRIS integration is activated. The scheduling start time is built with some randomization post integration activation to ensure the load on the processing is evenly
distributed for all organizations. There is not a way to control and  set a specific time for synchronization to start or trigger one on demand. The synchronization process has smarts built in it to handle failure by
rescheduling a new job for the failed organization and specific integrations that failed.

### <u>Input</u>

No input data.



### <u>Output</u>

| Name               | Type   |           Data Type           | Notes                                       | Mandatory |
|:-------------------|:-------|:-----------------------------:|:--------------------------------------------|:----------|
| Flow.statusCode    | Output | HTTP status code <br/>Integer | Less than 300 if success                    | Yes       |
| Flow.status        | Output |            String             | Set 'Complete' if success                   | Yes       |
| Flow.errorMsg      | Output |            String             | Message describing the error if not success | No        |
| Flow.emails        | Output |         String Array          | Emails configured in external HRIS system matching Genesys user. <br/>  Maximum of 2000 strings   | Yes |
| Flow.externalIds   | Output |         String Array          | Ids configured in external HRIS system for a matching Genesys user. <br/>Maximum of 2000 strings | Yes |
| Flow.emails1       | Output |         String Array          | Next bucket for 2000 emails                 | No        |
| Flow.externalIds1  | Output |         String Array          | Next bucket for 2000 externalIds            | No        |
| ..                 | ..     |              ..               | ..                                          |           |
| Flow.emails24      | Output |         String Array          | Next bucket for 2000 emails                 | No        |
| Flow.externalIds24 | Output |         String Array          | Next bucket for 2000 externalIds            | No        |

## <u>HRIS-Get-Timeoff-Types flow</u>

This flow receives a list of all time-off types from the HRIS. The time-off type consists of an ID/key, a name, and an optional secondary ID. Genesys Cloud workforce management sends this info back to the HRIS when it checks an agent's time-off balance or inserts or modifies time-off requests.

You can use the optional secondary ID if a single ID does not sufficiently identify a time-off type and its accrual rule in the HRIS. There's also an optional secondary ID that stores additional time-off type information.

When Genesys Cloud workforce management checks an agent's time-off balance or inserts or modifies a time-off request, it passes the secondary ID back to the HRIS.

### <u>Flow invocation context and details</u>

This flow is invoked in workforce management admin interface when creating a time off plan and associating the plan with external HRIS integration. When HRIS integration is chosen, the time off types associated in the external HRIS system is queried
for the admin to pick an external time off type to map to the given plan and activity codes. 

### <u>Input</u>

No input data.


### <u>Output</u>

| Name               | Type   |           Data Type           | Notes                                       | Mandatory |
|:-------------------|:-------|:-----------------------------:|:--------------------------------------------|:----------|
| Flow.statusCode    | Output | HTTP status code <br/>Integer | Less than 300 if success                    | Yes       |
| Flow.status        | Output |            String             | Set 'Complete' if success                   | Yes       |
| Flow.errorMsg      | Output |            String             | Message describing the error if not success | No        |
| Flow.ids           | Output |         String Array          | Ids of time off types configured in external HRIS system. <br/> Maximum of 2000 strings | Yes  |
| Flow.names         | Output |         String Array          | Names of time off types configured in external HRIS system. <br/> Count of names must match the ids specified. <br/> Maximum of 2000 strings | Yes  |
| Flow.id2s          | Output |         String Array          | Secondary Ids of time off types configured in external HRIS system <br/> Maximum of 2000 strings | No |


## <u>HRIS-Get-Balance flow</u>

An agent's time-off balances is received in this flow. This flow retrieves time off balances for all the requested dates and for extternal time off types requested.

### <u>Flow invocation context and details</u>

This flow is invoked from workorce management admin and agent time off request interfaces. When a time off request is requested, if the user in time off request is associated with a time off plan and the activity code chosen for the request
is associated with an HRIS integration, this workflow is invoked for time off request dates and external time off types. Dates passed are in YYYY-MM-DD format
and the balance is returned as of request day(s) for the given time off types.


### <u>Input</u>

| Name               | Type   |           Data Type           | Notes                                       | Mandatory |
|:-------------------|:-------|:-----------------------------:|:--------------------------------------------|:----------|
|Flow.inputDates     |Input   | String Array                  | Dates for which time off balance is requested in YYYY-MM-DD format. <br/> Maximum of 2000 strings | Yes       |
|Flow.inputTimeOffTypeIds|Input| String Array                 | External time off type ids for which time off balance is requested. <br/>Maximum of 2000 strings   | Yes       |
|Flow.agentId        |Input   | String                        | External HRIS agent Id for which the time off balance is requested. | Yes       |



### <u>Output</u>

| Name               | Type   |           Data Type           | Notes                                       | Mandatory |
|:-------------------|:-------|:-----------------------------:|:--------------------------------------------|:----------|
| Flow.statusCode    | Output | HTTP status code <br/>Integer | Less than 300 if success                    | Yes       |
| Flow.status        | Output |            String             | Set 'Complete' if success                   | Yes       |
| Flow.errorMsg      | Output |            String             | Message describing the error if not success | No        |
| Flow.balanceMinutesPerDay| Output |String Array             | Balances in minutes for a given date and external time off type id. <br/>Maximum of 2000 strings   | Yes   |
| Flow.timeOffTypeIds| Output |         String Array          | External time off type ids for the balances returned. <br>Maximum of 2000 strings  | Yes   |
| Flow.dates         | Output |         String Array          | Dates in YYYY-MM-DD format for the balances returned. <br>Maximum of 2000 strings   | Yes  |
| Flow.timeOffTypeId2s|Output |         String Array          | Secondary external time off type ids for the balances returned   | No   |



## <u>HRIS-Insert-TimeOff flow</u>

This flow inserts a new time-off request into the HRIS. On successful insertion and creation of time off on HRIS, the external timeOffRequestId after the creation is returned as output.
On successfull insertion, Genesys time off request will set to successful synchronization status. If workflow invovation results in an error, then the synchronization status is set to failed status.


### <u>Flow invocation context and details</u>

This flow is invoked if there is a time off request inserted from admin or agent interface and if user in the request is associated with time off plan and activity code in request is associated with HRIS integration. The workflow is invoked in attempt to synchronize and potentially
auto approve the request depending on configuration set on the associated time off plan and if there is sufficient balance on dates of time off request


### <u>Input</u>

| Name               | Type   |           Data Type           | Notes                                       | Mandatory |
|:-------------------|:-------|:-----------------------------:|:--------------------------------------------|:----------|
|Flow.agentId        |Input   | String                        | External HRIS agent Id for which the time off request is to be inserted | Yes       |
|Flow.dates          | Input  | String Array                  | List of dates for which time off is requested. <br/> For case of partial day time off requests, it will be in YYYY-MM-DD format of all the dates spanning the start and end times of the request. <br/> Maximum of 2000 strings | Yes       |
|Flow.startDateTimes | Input  | String                        | Applicable only to partial dat time off requests. The ISO datetime string in YYYY-MM-DDTHH:mm:ss format | No        |
|Flow.minDate        | Input  | String                        | Minimum date in YYYY-MM-DD format among spanned dates of time off request           | Yes       |
|Flow.maxDate        | Input  | String                        | Maximum date in YYYY-MM-DD format among spanned dates of time off request          | Yes       |
|Flow.timeOffStatus  | Input  | String                        | "APPROVED", "PENDING, "DENIED", "CANCELED"        | Yes       |
|Flow.timeOffTypeId  | Input  | String                        | External timeoff type id configured in HRIS                   | Yes       |
|Flow.payableMinutes | Input  | String Array| Payable minutes for each date in spanned dates of time off request         | Yes       |
|Flow.notes          | Input  | String                        | Optional notes for time off request          | No        |




### <u>Output</u>

| Name               | Type   |           Data Type           | Notes                                       | Mandatory |
|:-------------------|:-------|:-----------------------------:|:--------------------------------------------|:----------|
| Flow.statusCode    | Output | HTTP status code <br/>Integer | Less than 300 if success                    | Yes       |
| Flow.status        | Output |            String             | Set 'Complete' if success                   | Yes       |
| Flow.errorMsg      | Output |            String             | Message describing the error if not success | No        |
| Flow.timeOffRequestId           | Output |         String         | External timeOffRequestId created in HRIS                   | Yes       |


## <u>HRIS-Update-TimeOff flow</u>

This flow updates the details of time off request already associated with an external HRIS system. On successfull insertion, Genesys time off request will set to successful synchronization status. If workflow invovation results in an error, then the synchronization status is set to failed status.


### <u>Flow invocation context and details</u>

This flow is invoked if there is a time off request inserted from admin or agent interface and if time off request associated with external HRIS system that is synced previously is updated with details or if status changes. For example if a time off request is in synced with external HRIS system and is in
approved status and now it is changed to denied status, update workflow is invoked to update the external HRIS system to ensure the balances reflect the correct data.


### <u>Input</u>

| Name               | Type   |           Data Type           | Notes                                       | Mandatory |
|:-------------------|:-------|:-----------------------------:|:--------------------------------------------|:----------|
|Flow.agentId        |Input   | String                        | External HRIS agent Id for which the time off request is to be inserted | Yes       |
|Flow.dates          | Input  | String Array                  | List of dates for which time off is requested. <br/> For case of partial day time off requests, it will be in YYYY-MM-DD format of all the dates spanning the start and end times of the request. <br/> Maximum of 2000 strings | Yes       |
|Flow.startDateTimes | Input  | String                        | Applicable only to partial dat time off requests. The ISO datetime string in YYYY-MM-DDTHH:mm:ss format | No        |
|Flow.minDate        | Input  | String                        | Minimum date in YYYY-MM-DD format among spanned dates of time off request           | Yes       |
|Flow.maxDate        | Input  | String                        | Maximum date in YYYY-MM-DD format among spanned dates of time off request          | Yes       |
|Flow.timeOffStatus  | Input  | String                        | "APPROVED", "PENDING, "DENIED", "CANCELED"        | Yes       |
|Flow.timeOffTypeId  | Input  | String                        | External timeoff type id configured in HRIS                   | Yes       |
|Flow.payableMinutes | Input  | String Array| Payable minutes for each date in spanned dates of time off request         | Yes       |
|Flow.notes          | Input  | String                        | Optional notes for time off request          | No        |



### <u>Output</u>

| Name               | Type   |           Data Type           | Notes                                       | Mandatory |
|:-------------------|:-------|:-----------------------------:|:--------------------------------------------|:----------|
| Flow.statusCode    | Output | HTTP status code <br/>Integer | Less than 300 if success                    | Yes       |
| Flow.status        | Output |            String             | Set 'Complete' if success                   | Yes       |
| Flow.errorMsg      | Output |            String             | Message describing the error if not success | No        |
| Flow.timeOffRequestId           | Output |         String         | External timeOffRequestId associated in HRIS                   | Yes       |


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

Test your published flows with Genesys Cloud public API calls.

### Select test tool

There are two most convenient ways to make API calls to Genesys Cloud: Postman and [Genesys Cloud API Explorer](https://developer.genesys.cloud/devapps/api-explorer). Below you'll find an information about both tools usage.

As a client, you can also use your preferred method to call an API, for instance, curl.

#### Use Postman to test workflows

Postman is a popular third-party tool to make REST API requests. Genesys Cloud provides Postman collection among with environment files to simplify. Please, refer to [Genesys Cloud Developer Center - Postman](https://developer.genesys.cloud/platform/api/postman) for detailed information and authentication instructions.

In the sample Git repository, under postman_collection folder, you will find additional small collection, created specifically to test newly imported sample workflows, among with additional environment file. 

#### Use Genesys Cloud API Explorer to test workflows

[API Explorer](https://developer.genesys.cloud/devapps/api-explorer) provides convenient way to test your workflows with built-in authentication. For detailed usage guide, please refer to [API Explorer usage](https://developer.genesys.cloud/devapps/about/api-explorer).

* Click "Account Selection", allow browser storage and select your region. Select "Prompt to login" and login into your Organization.
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

To be able to test, for each imported workflow you'll need its unique ID. Open published Workflow in Architect and locate Id in your browser address line as shown below:
![alt text](images/architect_workflow_id.png)

### Trigger workflow execution

Every workflow execution could be triggered by [POST /api/v2/flows/executions](https://developer.genesys.cloud/routing/architect/#post-api-v2-flows-executions "Opens the POST /api/v2/flows/executions"). Below you can find examples of correct requests bodies for each workflow.

As a result of each request, you will receive an individual {flowExecutionId}, which will be used later.

Example output:
```
{
    "id": "{flowExecutionId}",
    "name": "sample name",
    "flowVersion": {
        "id": "1.0",
        "name": "1.0",
        "selfUri": "/api/v2/flows/${flow_id}/versions/1.0"
    },
    "selfUri": "/api/v2/flows/executions/{flowExecutionId}"
}
```

### Get flow execution result

Once {flowExecutionId} is received, you can check execution result by sending [GET /api/v2/flows/executions/{flowExecutionId}](https://developer.genesys.cloud/routing/architect/#get-api-v2-flows-executions--flowExecutionId- "Opens the GET /api/v2/flows/executions/{flowExecutionId}"). This request doesn't require any data or parameters.

### Examples of requests

Below you can find examples of requests and expected results for each sample workflow.


#### HRIS-Get-Agents-Flow

```
POST /api/v2/flows/executions
{
  "flowId": "${flow_id_get_agents}",
  "name": "try_get_agents"
}
```

Example of successful execution:

```

    "id": {flowExecutionId},
    "name": "try_get_agents",
    "flowVersion": {...}
    "dateLaunched": "2025-03-26T19:50:17.823Z",
    "status": "COMPLETED",
    "dateCompleted": "2025-03-26T19:50:18.529Z",
    "completionReason": "Success",
    "outputData": {
        "Flow.status": "Complete",
        "Flow.statusCode": "200",
        "Flow.externalIds1": [],
        "Flow.emails1": [],
        "Flow.externalIds2": [],
        "Flow.emails2": [],
        "Flow.externalIds3": [],
        "Flow.emails3": [],
        "Flow.emails": [
         "user.one@companydomain.com",
         "user.two@companydomain.com",
         "user.three@companydomain.com",
        ],
        "Flow.errorMsg": null,
        "Flow.externalIds": [
            "313",
            "332",
            "452"
        ]
    },
    "selfUri": "/api/v2/flows/executions/{flowExecutionId}"
}
```

#### HRIS-Get-Timeoff-Types flow

```
POST /api/v2/flows/executions
{
  "flowId": "${flow_id_get_timeoff_types}",
  "name": "try_get_timeoff_types"
}
```

Example of successful execution:
```
{
    "id": "{flowExecutionId}",
    "name": "try_get_timeoff_types",
    "flowVersion": {...}
    "dateLaunched": "2025-03-26T20:14:38.338Z",
    "status": "COMPLETED",
    "dateCompleted": "2025-03-26T20:14:38.905Z",
    "completionReason": "Success",
    "outputData": {
        "Flow.statusCode": "200",
        "Flow.ids": [
            "83",
            "90",
            "85"
        ],
        "Flow.status": "Complete",
        "Flow.errorMsg": null,
        "Flow.names": [
            "Open Time Off",
            "Study Day",
            "Budgeted days"
        ],
        "Flow.id2s": []
    },
    "selfUri": "/api/v2/flows/executions/{flowExecutionId}"
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
    
  },
  "name": "try_get_balance"
}
```
Example of successful execution:
```
{
    "id": {flowExecutionId},
    "name": "try_get_balance",
    "flowVersion": {...},
    "dateLaunched": "2025-03-26T20:33:25.890Z",
    "status": "COMPLETED",
    "dateCompleted": "2025-03-26T20:33:26.959Z",
    "completionReason": "Success",
    "outputData": {
        "Flow.balanceMinutesPerDay": [
            "9600",
            "480"
        ],
        "Flow.statusCode": "200",
        "Flow.timeOffTypeIds": [
            "83",
            "84"
        ],
        "Flow.timeOffTypeId2s": [],
        "Flow.status": "Complete",
        "Flow.errorMsg": null,
        "Flow.dates": [
            "2025-06-01",
            "2025-06-01"
        ]
    },
    "selfUri": "/api/v2/flows/executions/{flowExecutionId}"
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
  },
  "name": "try_insert_timeoff"
}
```
Example of successful execution:
```
{
    "id": "{flowExecutionId}",
    "name": "try_insert_timeoff",
    "flowVersion": {...}
    "dateLaunched": "2025-03-26T22:13:16.340Z",
    "status": "COMPLETED",
    "dateCompleted": "2025-03-26T22:13:17.608Z",
    "completionReason": "Success",
    "outputData": {
        "Flow.timeOffRequestId": "4828",
        "Flow.statusCode": "200",
        "Flow.error": null,
        "Flow.status": "Complete",
        "Flow.errorMsg": null
    },
    "selfUri": "/api/v2/flows/executions/{flowExecutionId}"
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
  },
  "name": "try_update_timeoff"
}
```
Example of successful execution:
```
{
    "id": "{flowExecutionId}",
    "name": "try_update_timeoff",
    "flowVersion": {...}
    "dateLaunched": "2025-03-26T22:14:27.487Z",
    "status": "COMPLETED",
    "dateCompleted": "2025-03-26T22:14:28.847Z",
    "completionReason": "Success",
    "outputData": {
        "Flow.timeOffRequestId": "4829",
        "Flow.statusCode": "200",
        "Flow.error": null,
        "Flow.status": "Complete",
        "Flow.errorMsg": null
    },
    "selfUri": "/api/v2/flows/executions/{flowExecutionId}"
}
```

## Additional resources

* [wfm-hris-blueprint repository](https://github.com/GenesysCloudBlueprints/wfm-hris-blueprint "Opens the wfm-hris-blueprint repository") in GitHub.
