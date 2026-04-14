# User Experience

## Pages

### ADD Acubiz Setup
- Type: Card
- Purpose: Maintain the singleton journal-target setup for Acubiz imports.
- Target user: finance or integration administrator.
- Source entity: ADD Acubiz Setup.

#### Layout contract
| Section | Fields / content | Purpose |
|---------|------------------|---------|
| General | journal template name, journal batch name | defines where imported transactions will be created |

#### Conditional behavior
- The page auto-creates the setup record on first open.
- Insert and delete are disallowed from the page.

### ADD Azure Configs
- Type: List
- Purpose: Maintain one Azure configuration per integration scenario.
- Target user: integration administrator.
- Source entity: ADD Azure Config.

#### Layout contract
| Section | Fields / content | Purpose |
|---------|------------------|---------|
| Main repeater | integration type, storage account, container, source subdirectory, archive subdirectory, masked storage key, subject, document category, full-name filter, GLN-check flag | configures storage access and scenario-specific routing behavior |

#### Conditional behavior
- When a stored secret exists, the password field shows a masked placeholder.
- Editing the password field replaces the isolated secret and commits immediately.

### ADD Acubiz Godkendere
- Type: List
- Purpose: View approver records from a shared table and run employee-related import/export actions.
- Target user: HR/integration administrator.
- Source entity: external shared approver table.

#### Layout contract
| Section | Fields / content | Purpose |
|---------|------------------|---------|
| Main repeater | approver name, payroll number | review approver mapping data |

#### Actions
| Action | Availability | Effect |
|--------|--------------|--------|
| Import Employees | Page action | runs the employee import xmlport |
| Acubiz Employee List | Page action | runs the employee-export transformation xmlport |

### General Journal page extension
- Type: Page extension
- Purpose: Add a direct action for importing Acubiz transactions into the current journal context.
- Target user: finance user working in General Journal.

#### Actions
| Action | Availability | Effect |
|--------|--------------|--------|
| Import Acubiz Transactions | available on General Journal | runs the Acubiz import xmlport with the current journal context and refreshes the page |

## Navigation
- Administrators maintain setup from dedicated setup pages.
- Finance users can trigger import directly from the General Journal page.
- Approver/employee-related export functions are available from the approver list page.

## UX posture
The app is utilitarian and admin-focused. Its UI is primarily configuration and launch surfaces for file-processing workflows, not rich operational dashboards.