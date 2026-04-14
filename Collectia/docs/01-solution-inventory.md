# Solution Inventory

## Source Inventory Summary
| Object kind | Count | Notes |
|-------------|-------|-------|
| Tables | 1 | Setup table |
| Table extensions | 2 | Issued Reminder Header and Reminder Level |
| Pages | 2 | Setup card and case-creation input dialog |
| Page extensions | 2 | Issued Reminder and Reminder Levels |
| Codeunits | 2 | SOAP WS client and event subscriber |
| Enums | 1 | Case-start mode |
| Permission sets | 0 | Missing — must be added |
| Test objects | 0 | Missing — must be added |

## Domain Breakdown
| Domain | Purpose | Included objects |
|--------|---------|------------------|
| Setup | Collectia connection and behavior configuration | setup table, setup page |
| CaseCreation | Create collection cases at Collectia from issued reminders | WS codeunit, input dialog page, enum |
| DocumentUpload | Upload invoice PDFs and document metadata to Collectia cases | within WS codeunit |
| ReminderIntegration | Track collection status on reminders and trigger automatic sending | table extensions, page extensions, event subscriber |

## Object Inventory
| Object kind | Object name | Domain | Purpose | Required for rebuild | Depends on |
|-------------|-------------|--------|---------|----------------------|------------|
| Table | Collectia Setup | Setup | Stores credentials, endpoint, defaults, and behavior flags | Yes | — |
| Page | Collectia Setup | Setup | Maintains setup record | Yes | setup table |
| Enum | Begin Case in Default | CaseCreation | INK (inkasso) vs RYK (rykkerservice) mode selection | Yes | setup table, input dialog |
| Codeunit | Collectia WS | CaseCreation + DocumentUpload | All SOAP interactions: login, create case, send documents, activate, logout, logging | Yes | setup, BC reminders/invoices, external service |
| Page | Collectia Input | CaseCreation | Dialog for manual case creation — collects summons text and mode | Yes | setup, enum |
| Codeunit | Reminder Subscribers | ReminderIntegration | Event subscriber for automatic case creation on reminder issue | Yes | WS codeunit, reminder level extension |
| Table extension | Issued Reminder Header extension | ReminderIntegration | Adds collection-sent flag and job number to issued reminders | Yes | BC Issued Reminder Header |
| Table extension | Reminder Level extension | ReminderIntegration | Adds send-to-collection flag per reminder level | Yes | BC Reminder Level |
| Page extension | Issued Reminder extension | ReminderIntegration | Shows collection fields and adds manual send/open actions | Yes | BC Issued Reminder page |
| Page extension | Reminder Levels extension | ReminderIntegration | Shows send-to-collection checkbox | Yes | BC Reminder Levels page |

## Non-code Assets
| Asset | Purpose | Rebuild relevance |
|-------|---------|-------------------|
| app.json | manifest | High — but needs complete modernization |
| nubiz.dk.square.png (referenced but missing) | logo | Low |
