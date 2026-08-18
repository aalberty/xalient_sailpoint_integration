# Xalient Identity Connect

## Overview

Xalient Identity Connect is a ServiceNow application that bridges ServiceNow catalog requests with SailPoint Identity Security Cloud (ISC) provisioning. It enables requesters to order access from ServiceNow catalog items, map SailPoint entitlement data into ServiceNow business applications, and drive approvals and provisioning workflows through ISC.

This application is designed to support scenarios where:

- ServiceNow catalog items are used to request access
- SailPoint access items and entitlements are represented as catalog-ready requestable items
- approvals are handled through ServiceNow workflows and fallback policies
- provisioning is executed against SailPoint ISC via REST-based integration
- provisioning status is monitored and surfaced back into ServiceNow

## What the application does

At a high level, the application provides the following capabilities:

1. Catalog-driven access requests
   - Includes ServiceNow catalog items such as access requests and ownership-change scenarios.
   - Uses Catalog Item Config records to define how each catalog item behaves.

2. SailPoint access item modeling
   - Stores SailPoint access item information in the custom table for ISC Access Items.
   - Supports mapping each access item to a ServiceNow business application.
   - Tracks metadata such as SailPoint ID, source name, entitlement value, object type, owner, requestability, and approval scheme.

3. Approval orchestration
   - Supports configurable approval schemes per catalog item or access item.
   - Includes fallback approval behavior for cases where an approver cannot be resolved.
   - Supports approval rules through a dedicated table and script-based logic.

4. Provisioning integration with SailPoint ISC
   - Uses a REST message for ISC called “SailPoint ISC”.
   - Includes dedicated REST message functions for actions such as:
     - access request creation
     - token retrieval
     - identity and entitlement lookups
     - workflow execution
     - status monitoring
     - lifecycle state changes

5. Task and workflow automation
   - Supports provisioning tasks and post-provisioning tasks through the Catalog Item Tasks table.
   - Uses ServiceNow Flow Designer / Hub flow artifacts to submit and monitor provisioning requests.

## Main components

### 1. Custom tables

The application introduces several custom tables:

- Catalog Item Config
  - Maps catalog items to business applications and configures approval/provisioning behavior.
  - Supports fields such as approval scheme, fallback approval scheme, provisioning tasks, and application filter visibility.

- ISC Access Items
  - Represents SailPoint access items that can be requested through ServiceNow.
  - Includes a reference to business applications and SailPoint metadata.

- Catalog Item Tasks
  - Defines tasks that should be created before, during, or after provisioning.

- Approval Rule
  - Stores script-based approval logic.

- SailPoint Access Item Staging
  - An import-set staging table used for ingestion of SailPoint access item data.

### 2. Integration artifacts

The app includes:

- REST Message: SailPoint ISC
- REST Message Functions for ISC endpoints
- OAuth-related configuration entities
- Flow Designer / Hub Flow for provisioning request submission and status polling
- System properties for tenant, domain, credentials, timeouts, and fallback behavior

### 3. Service Catalog artifacts

The update set also contains catalog item definitions and variable sets, including examples such as:

- Access Item Ownership Change
- CyberArk Access Request
- Peloton Application Access
- SailPoint Access Request variable set / request experience

These catalog items are likely intended to surface the entitlement request flow in a user-friendly way.

## Request flow

A typical end-to-end request flow looks like this:

1. A user submits a ServiceNow catalog request.
2. The catalog item and its configuration determine:
   - which access items can be selected
   - whether an application filter is shown
   - which approval scheme applies
   - which provisioning tasks are triggered
3. The request is evaluated against approval rules and fallback logic.
4. The provisioning request is submitted to SailPoint ISC via the integration layer.
5. ServiceNow monitors the provisioning status and updates the request or tasks accordingly.

## Configuration required

Before using the application in a target instance, the following should be configured:

### ServiceNow configuration

- Import the update set into the target ServiceNow instance.
- Activate/install the application and any required dependencies.
- Ensure the application scope is available and the relevant roles are assigned.
- Review catalog item visibility, categories, and user criteria.

### ISC integration configuration

The app includes a dedicated configuration category named “Xalient ISC Configuration”. Configure these system properties:

- x_xal_idconnect.isc_tenant
  - Your SailPoint ISC tenant name (for example, the first part of the ISC URL).

- x_xal_idconnect.domain
  - The ISC domain portion of the URL, such as identitynow.com.

- x_xal_idconnect.isc_client_id
  - OAuth client ID for the ISC integration.

- x_xal_idconnect.isc_client_secret
  - OAuth client secret for the ISC integration.

- x_xal_idconnect.approval_fallback_group
  - Group used when an approval cannot be resolved automatically.

- x_xal_idconnect.sc_task_assignment_group
  - Default assignment group for Service Catalog tasks.

- x_xal_idconnect.incident_assignment_group
  - Default assignment group for incidents created by the integration.

- x_xal_idconnect.max_status_checks
  - Maximum number of provisioning status checks.

- x_xal_idconnect.status_check_interval
  - Time interval between provisioning checks.

### SailPoint data preparation

To make access items available in ServiceNow:

- Load or sync SailPoint access item data into the staging table.
- Review and transform the imported rows as required.
- Promote valid rows into the ISC Access Items table.
- Set the relevant fields such as:
  - Name
  - Description
  - SailPoint ID
  - Source name / source ID
  - Entitlement value
  - Business Application
  - Use in Catalog Items
  - Requestable / Active / Privileged flags

### Catalog mapping

For each catalog item that should support this flow:

- Create or update a Catalog Item Config record.
- Associate the correct catalog item.
- Choose the approval scheme.
- Choose whether the request should allow single or multiple access item selection.
- Add relevant business applications.
- Link provisioning tasks if needed.

## Suggested operational workflow

A practical implementation workflow is:

1. Configure the ISC integration properties.
2. Populate SailPoint access items into the staging/import process.
3. Create or review Catalog Item Config records for each catalog item.
4. Map access items to business applications.
5. Test a low-risk catalog request.
6. Validate approvals, provisioning submission, and status updates.
7. Adjust assignment groups, approval rules, and task definitions as needed.

## Notes and best practices

- Treat the Catalog Item Config records as the primary control point for request behavior.
- Keep approval logic simple to start, then expand as business rules become clearer.
- Use the fallback approval group carefully to avoid requests getting stuck.
- Validate that the named system properties exist and values are accurate before testing.
- Because this is an integration-heavy app, test in a non-production instance first.
- Review the REST message definition and flow outputs carefully if provisioning fails or status is not updated.

## Expected outcomes

Once configured, the application should allow you to:

- request access from ServiceNow using catalog-driven forms
- surface SailPoint access items in a ServiceNow-friendly model
- link those access items to business applications
- route approvals based on configurable rules
- trigger provisioning through SailPoint ISC
- track request outcomes and provisioning status

## Useful follow-up areas

If you want to extend or harden this implementation further, the next useful areas to inspect are:

- approval rule definitions
- task assignment and incident assignment groups
- the flow(s) that submit and monitor provisioning requests
- the data import process for SailPoint access items
- catalog item variables and UI policies that govern form behavior

## Summary

This application is essentially an identity request and provisioning bridge between ServiceNow and SailPoint ISC. Its value is in turning SailPoint entitlement information into catalog-driven ServiceNow requests, then orchestrating approvals and provisioning through a configurable integration layer.
