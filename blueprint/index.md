# wfm-hris-blueprint (beta)

The integration allows Genesys Cloud to get employee time-off balances from HRIS and provide that information in WFM. When time-off request is created or updated in Genesys Cloud WFM, the integration submits the request to HRIS for tracking and verifying against the balance.

The integration is congifured and running within Genesys Cloud environment. It is based on Genesys Archtect Flows and uses Genesys Cloud Data Actions to call HRIS APIs

The beta 1 is intended for testing. The blueprint explains how to configure Flows and Data Actions, so they can be set up and tested before the functionality can be used by Genesys Cloud Workforce Management product. Genesys Cloud Workforce Management will not be using integration functionality withing the scope of beta 1.

## Solution Components

- Genesys Cloud - The Genesys cloud-based contact center platform. The following components are used;
  - Genesys Cloud Integrations and Data Actions
  - Genesys Cloud Architect
- 3rd party HRIS, supporting JSON REST API. The integration requires external API access from Genesys Cloud to the HRIS.

## Requirements

### Specialized knowledge

- Understanding of HRIS time-off sub-system and its API
- Ability to design and create flows with Genesys Cloud Architect.
- Understanding of JSON REST API, authentication and Genesys Data Actions.
- Genesys Cloud API for flows testing through API

### HRIS account with API access

HRIS account needs to be configured to provide a limited and secured access to its data through JSON based REST APIs. 

- WFM HRIS Integration needs the following API access
  - Get a list of configured time-off types
  - Get a list of agents, for whom the integration is intended
    - Agent unique key or id withing HRIS
    - Agent email to map with the corresponding agent object instance in Genesys Cloud
  - Retrieve agent time-off balance on requested dates, for specific time-off types
  - Insert time-off request for the agent
  - Modify/delete previously inserted agent time-off request

### Genesys Cloud account

- A Genesys Cloud license. One of listed below is required
  - Genesys Cloud CX 3
  - Genesys Cloud CX 1 WEM Upgrade 2
  - Genesys Cloud CX 2 WEM Upgrade 1

For more information, see [Genesys Cloud Pricing](https://www.genesys.com/pricing) in the Genesys website.

### Genesys Cloud user with appropriate permissions

The user needs appropriate Integrations and Architect permissions.
For more information, see [Roles and permissions overview](https://help.mypurecloud.com/articles/about-roles-permissions/) in the Genesys Cloud Resource Center.

To test Architect flows using Genesys Cloud API, the user user will need OAuth permissions to setup the client.

## Integration architecture

The integration is based on Architect workflows. The execution of those workflows is intended to be started by Genesys Cloud WFM.
Workflows in turn use one or more Data Actions to communicate with HRIS. Workflow can make multiple invocations of one or more Data Actions to get or modify HRIS data. Flow can also implement additional logic to manipulate the data in desired way.
Data Actions use credentials stored in Integration configuration to get authorization from HRIS to access specified routed of HRIS JSON REST API.

### Flow types

The integration should have the following flows

#### Synchronize agents

The flow gets all agents from HRIS. It needs to get agent HRIS id/key together with their email.
WFM is supposed to map an agent with a corresponding object in Genesys Cloud based on email and store HRIS agent id for future communications with HRIS to get balances and insert time-off requests.

This workflow is optional as there are means in WFM to enter HRIS agent id manually.

**Input:** none

**Output:** A list of agents consisting of id and email

#### Get Time-off Types

The flow gets all time-off types from HRIS. Time-off type information consists of its id/key, name and optional secondary id.
WFM will pass this info back to HRIS, when checking agent time-off balance or inserting/modifying time-off request.
An optional secondary id can be used in integrations, when a single id is not enough to identify time-off type and, say, its accrual rule in HRIS. It can also be used to store any additional information about time-off type that integration chooses.
If secondary id is returned, WFM will also pass it back to HRIS, when checking agent time-off balance or inserting/modifying time-off request.

**Input:** none

**Output:** A list of time-off types

#### Checking agent time-off balance

The flow get the time-off balance information for a single agent.
With one call, it can retrieve balance for multiple time-off type and multiple days.

**Input:** Agent id, a list of dates and a list of time-off types

**Output:** A list of balances for dates and time-off types

#### Inserting new time-off request

The flow inserts a new time-off request into HRIS. WFM calls this flow to propagate time-off into into HRIS.
WFM will propagate only APPROVED time-off requests.
If agents cannot exceed certain balance thresholds and the flag to override the balance is false, this flow should be checking that threshold as well, for which it may need to invoke a separate Data Action.

**Input:** Agent id, a flag to override the balance, earliest time-off request date, latest time-off request date, average amount of time-off minutes (hours) per time-off day, list of time-off days containing date and an amount of time-off minutes per day.

**Output:** An id of newly created HRIS time-off request.

#### Updating existing time-off request

The flow updates HRIS time-off information, when previously propagated to HRIS time-off request changes in WFM. The flow is similar to the inserting flow, with the difference that 

**Input:** Agent id, HRIS time-off request id, a flag to override the balance, new time-off status, earliest time-off request date, latest time-off date, average amount of time-off minutes (hours) per time-off day, list of time-off days containing date and an amount of time-off minutes per day.

**Output:** An id of HRIS time-off request. Resulting id could be different that one in the input as flow or HRIS itself may remove old and insert new time-off request.

## Deployment Steps

### Download the repository containing project files

- Clone the repository

NOTE: This step is needed only, if you intend to use provided Workflow and Data Action examples. The examples are for integration with BambooHR HRIS. If integrating with another HRIS, the examples will require modifications.

### Create the Integration

- In Genesys Cloud, go to Admin > Integrations > Integrations and install the Integration of type "Web Services Data Actions"
- Once properties of new Integration opens
  - In "Details" section you may change the name of your integration
  - Go to "Configuration" tab, "Credentials" section and configure credentials to access HRIS.

### Create Data Actions

Data Actions make JSON API calls to HRIS. Each Data Action is configured for specific API route and is set up to retrieve pass to and retrieve from HRIS specific data.  

Data Action configuration depends on API of specific HRIS. 

- In Genesys Cloud, go to Admin > Integrations > Actions and create or import a new Action
  - Associate the Action with the Integration created in the previois step
  - Go to "Setup" tab and configure "Contracts" and "Configuration" sections
    - Note that "Contracts" section has input and output contracts that are going to be used by your workflow.  

### Create Workflows

Five workflows need to be created, one of each type documented above.

- In Genesys Cloud, go to Admin > Architect
  - New Genesys Cloud Architect tab opens
- Choose "Workflow" type in "Flows" selector
- "Add" new flow
  - You may import an example, once an empty flow is created
- "Save" and "Publish" the flow, once it is completed

For more information about Architect and flows, use Architect online help. Also, Genesys Resource Center has an article about [working with workflows](https://help.mypurecloud.com/articles/work-with-workflows/)  

## Testing workflows using Genesys CLoud API

Once workflows are saved and published, they can be tested using Genesys Cloud Public API calls.
'curl' tool can be used as a client to call API, but there are many other options

- First, the client needs to authenticate and get a bearer token to be able to call an API. Click the link for more information about [API client configuration, authentication and authorization](https://developer.genesys.cloud/authorization/platform-auth/use-authorization-code).
- Use API calls to view, execute flow and get flow execution results. See more information about API on [developer center](https://developer.genesys.cloud/devapps/api-explorer). Select "Architect" category.
  - GET /api/v2/flows/ - to check the configured flows
  - POST /api/v2/flows/executions - to pass input parameters and start flow execution
  - GET /api/v2/flows/executions/{flowExecutionId} - to check flow execution status and get results
