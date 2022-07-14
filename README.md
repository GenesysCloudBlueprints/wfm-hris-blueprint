# wfm-hris-blueprint

This is a blueprint to demonstrate how to integrate Genesys Cloud Workforce Management (WFM) with 3rd-party Human Resources Information Systems (HRIS).
The integration allows Genesys Cloud to get employee time-off balances from HRIS and provide that information in WFM. When time-off request is created or updated in Genesys Cloud WFM, the integration submits the request to HRIS for tracking and verifying against the balance.

## Solution Components

- Genesys Cloud - The Genesys cloud-based contact center platform. Genesys Cloud is the platform for WFM and Architect applications and Integrations management.
  - Genesys Cloud WFM application
  - Genesys Cloud Integrations and Data Actions
  - Genesys Cloud Architect
    - Access to "workflow" type Architect flows.
- 3rd party HRIS, supporting JSON REST API. The integration requires external API access to the HRIS. 
  - API to retrieve time-off types from HRIS, which are going to be used to retrieve agent time-off balance and submit their time-off requests
  - API to get agent time-off balance
  - API to submit agent time-off request
  - API to update existing agent request
  - (optionally) API to retrieve agent unique id within HRIS and their email address

## Requirements

### Specialized knowledge

- Ability to design and create flows with Genesys Cloud Architect.
- Understanding of JSON REST API, authentication and Genesys Data Actions.
- Understanding of HRIS time-off sub-system and its API

## Deployment Steps

## Additional Resources