---
title: Integrate Genesys Cloud with an HRIS system to sync time-off requests and time-off balances from Genesys Cloud WFM
author: Vidas Placiakis
indextype: blueprint
icon: blueprint
image: images/hris_integration_overview.png
category: 12
summary: |
  This Genesys Cloud Developer Blueprint integrates Genesys Cloud with an HRIS to retrieve employee time-off balances and provide that information in Genesys Cloud workforce management. When a time-off request is created or updated in Genesys Cloud workforce management, the integration inserts the request to the HRIS for tracking and verification against the employee's available balance.
---
:::{"alert":"primary","title":"About Genesys Cloud Blueprints","autoCollapse":false} 
Genesys Cloud blueprints were built to help you jump-start building an application or integrating with a third-party partner. 
Blueprints are meant to outline how to build and deploy your solutions, not a production-ready turn-key solution.
 
For more information about Genesys Cloud blueprint support and practices, see our Genesys Cloud blueprint [FAQ](https://developer.genesys.cloud/blueprints/faq "Opens the Blueprint FAQ") sheet.
:::

This Genesys Cloud Developer Blueprint integrates Genesys Cloud with an HRIS to retrieve employee time-off balances and provide that information in Genesys Cloud workforce management. When a time-off request is created or updated in Genesys Cloud workforce management, the integration inserts the request into the HRIS for tracking and verification against the employee's available balance.

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

### HRIS-Get-Agents flow

This HRIS provides a list of all agent records, including their corresponding IDs/keys and email addresses. Genesys Cloud workforce management requires this data to retrieve time-off balances and insert time-off requests for an agent.

This flow is optional. Alternatively, you could manually enter an agent's HRIS ID into Genesys Cloud workforce management.

### HRIS-Get-Timeoff-Types flow

This flow receives a list of all time-off types from the HRIS. The time-off type consists of an ID/key, a name, and an optional secondary ID. Genesys Cloud workforce management sends this info back to the HRIS when it checks an agent's time-off balance or inserts or modifies time-off requests.

You can use the optional secondary ID if a single ID does not sufficiently identify a time-off type and its accrual rule in the HRIS. There's also an optional secondary ID that stores additional time-off type information.

When Genesys Cloud workforce management checks an agent's time-off balance or inserts or modifies a time-off request, it passes the secondary ID back to the HRIS.

### HRIS-Get-Balance flow

An agent's time-off balances is received in this flow. Multi-day and multi-time-off balance information are retrieved.

### HRIS-Insert-TimeOff flow

This flow inserts a new time-off request into the HRIS. The request contains the following information:

* Agent ID
* A flag (true/false) to override the balance threshold
* Earliest time-off request date
* Latest time-off request date
* Average amount of time-off minutes (hours) per time-off day
* List of time-off days containing a date
* Amount of time-off minutes per day

:::primary
**Note**
Time-off requests are only propagated if approved by Genesys Cloud workforce management.
:::

This flow should check if agents cannot exceed certain balance thresholds in the HRIS and the flag to override the threshold is set to false. If this is the case, data actions may have to be invoked separately.

### HRIS-Update-TimeOff flow

The corresponding information in WFM changes when a time-off record in the HRIS is updated. This request contains the following information:

* Agent ID
* HRIS time-off request ID
* A flag (true/false) to override the balance threshold
* New time-off status
* Earliest time-off request date
* Latest time-off request date
* Average amount of time-off minutes (hours) per time-off day
* List of time-off days containing a date
* Amount of time-off minutes per day

:::primary
**Note** HRIS time-off request IDs are updated or replaced based on whether an existing record is updated or replaced.
:::

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
5. Click **Save**.

## Test the workflows with the Genesys Cloud API

Test your published flows with Genesys Cloud public API calls. As a client, you can use curl or your preferred method to call an API.

1. Authenticate your client and get a bearer token to call an API. For more information, [Grant - Authorization Code](https://developer.genesys.cloud/authorization/platform-auth/use-authorization-code "Goes to the Grant - Authorization Code page").
2. View a flow, execute a flow, and obtain execution results using the following API calls:
  * [GET /api/v2/flows/{flowId}](https://developer.genesys.cloud/routing/architect/#get-api-v2-flows--flowId- "Opens the GET /api/v2/flows/{flowId}") to check a configured flow.
  * [POST /api/v2/flows/executions](https://developer.genesys.cloud/routing/architect/#post-api-v2-flows-executions "Opens the POST /api/v2/flows/executions") to pass input parameters and start flow execution.
  * [GET /api/v2/flows/executions/{flowExecutionId}](https://developer.genesys.cloud/routing/architect/#get-api-v2-flows-executions--flowExecutionId- "Opens the GET /api/v2/flows/executions/{flowExecutionId}") to check flow execution status and get results.

  For more information, see the [Architect APIs](https://developer.genesys.cloud/routing/architect/ "Opens the Architect APIs") in the Genesys Cloud Developer Center.

## Additional resources

* [wfm-hris-blueprint repository](https://github.com/GenesysCloudBlueprints/wfm-hris-blueprint "Opens the wfm-hris-blueprint repository") in GitHub.
