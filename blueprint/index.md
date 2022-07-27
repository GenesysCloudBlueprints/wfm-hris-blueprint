# wfm-hris-blueprint (beta)

The integration allows Genesys Cloud to get employee time-off balances from HRIS and provide that information in WFM. When time-off request is created or updated in Genesys Cloud WFM, the integration submits the request to HRIS for tracking and verifying against the balance.

The integration is congifured and running within Genesys Cloud environment. It is based on Genesys Archtect flows and uses Genesys Cloud Data Actions to call HRIS APIs

## Solution Components

- Genesys Cloud - The Genesys cloud-based contact center platform. Genesys Cloud is the platform for WFM and Architect applications and Integrations management.
  - Genesys Cloud Integrations and Data Actions
  - Genesys Cloud Architect
    - Access to "workflow" type Architect flows.
- 3rd party HRIS, supporting JSON REST API. The integration requires external API access from Genesys Cloud to the HRIS. 
  - API to retrieve time-off types from HRIS, which are going to be used to retrieve agent time-off balance and submit their time-off requests
  - API to get agent time-off balance
  - API to submit agent time-off request
  - API to update existing agent request
  - (optionally) API to retrieve agent unique id within HRIS and their email address

QUESTION: we do not say that WFM is component for beta, correct?

## Requirements

### Specialized knowledge

- Ability to design and create flows with Genesys Cloud Architect.
- Understanding of JSON REST API, authentication and Genesys Data Actions.
- Understanding of HRIS time-off sub-system and its API

### Genesys Cloud account

- A Genesys Cloud license.
  - A product license with the ability to add "workflow" type of flow in Architect
  - A user with a role to access Genesys Could Architect, Integrations and Data Actions

QUESTION: what exactly product do we suggest ??

### HRIS account for API access

- API needs to
  - Get a list of configured time-off types
  - Get a list of agents
    - Agent unique key or id withing HRIS
    - Agent email to map with the corresponding agent object instance in Genesys Cloud
  - Retrieve agent time-off balance
  - Insert time-off request for the agent
  - Modify/delete time-off request for the agent

## Deployment Steps

### Download the repository containing project files

- Clone the repository

NOTE: This step is needed only, if you intend to use provided Workflow and Data Action examples. The examples are for integration with BambooHr HRIS. If integrating with another HRIS, the examples will require modifications.

### Create the Integration

- In Genesys Cloud, go to Admin > Integrations > Integrations and install the  Integration of type "Web Services Data Actions"
- Once properties of new Integration opens
  - In "Details" section you may change the name of your integration
  - Go to "Configuration" tab, "Credentials" section and configure credentials to access HRIS.

### Create Data Actions

Data Actions make JSON API calls to HRIS. Each Data Action is configured for specific API route and is set up to retrieve pass to and retrieve from HRIS specific data.  

Data Action configuration depends on API of specific HRIS. The examples provided with the blueprint are for BambooHr.

- In Genesys Cloud, go to Admin > Integrations > Actions and create or import a new Action
  - Associate the Action with the Integration created in the previois step
  - Go to "Setup" tab and configure "Contracts" and "Configuration" sections
    - Note that "Contracts" section has input and output contracts that are going to be used by your workflow.  

### Create Workflows

## Additional Resources

TBD.