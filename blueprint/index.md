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