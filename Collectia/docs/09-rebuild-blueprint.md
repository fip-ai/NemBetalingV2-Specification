# Rebuild Blueprint

## App manifest blueprint
- App name: Collectia (replacing "Inkasso")
- Test app name: Collectia Test
- App role: debt-collection integration with Collectia external service
- Dependencies: BC base application only
- Runtime/target posture: modern Cloud target, current runtime

## Reconstruction order
1. Build setup table with isolated credential storage.
2. Build case-mode enum with proper captions.
3. Build HTTP/SOAP transport interface and production implementation.
4. Build case-creation management codeunit (pure logic, delegating transport to interface).
5. Build document-upload management codeunit.
6. Build reminder table extensions with collection tracking fields.
7. Build setup page, input dialog, page extensions on issued reminders and reminder levels.
8. Build event subscriber for automatic case creation.
9. Build permission set.
10. Build paired test app with transport stub and full acceptance suite.

## Object blueprint by domain

### Setup
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Table | Collectia Setup | singleton config with credential isolation |
| Page | Collectia Setup | admin card with masked credential display |

### CaseCreation
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Enum | Case Start Mode | INK/RYK with proper captions |
| Interface | SOAP transport abstraction | Send method accepting request and returning response |
| Codeunit | SOAP transport implementation | HTTP POST with content-type and timeout handling |
| Codeunit | Collectia case management | login, create case, logout, with validation and error handling |
| Page | Case creation input dialog | collects summons text and mode |

### DocumentUpload
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Codeunit | Collectia document management | PDF generation, document upload, metadata submission, type mapping |

### ReminderIntegration
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Table extension | Issued Reminder Header | sent flag and case number fields |
| Table extension | Reminder Level | automatic-send flag |
| Page extension | Issued Reminder | collection fields and send/open/log actions |
| Page extension | Reminder Levels | automatic-send checkbox |
| Codeunit | Reminder event subscriber | OnAfterIssueReminder subscriber for automatic case creation |

### Security
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Permission set | Collectia | grants all production object access |
| Permission set | Collectia Test | grants test execution |

## Mandatory behaviors
- Reminder can only be sent once to Collectia (duplicate prevention via case-number check).
- Summons text is required for all case creation.
- Danish customers are validated for VAT and postal code length.
- Foreign customers use a different payload template with UDL type.
- Customer blocking is conditional on setup flag.
- Document type is mapped from BC document types to Collectia amount-type codes.
- Additional fees are aggregated into a single submission.
- Activity logging is optional and controlled by setup.

## Mandatory tests for rebuilt solution
- All acceptance criteria from `08-test-specification.md`.
- Transport stub must replace real HTTP calls in all tests.
- Both manual and automatic case creation paths must be tested.
- All validation rules must have positive and negative tests.
- Document type mapping must be tested for every BC document type.

## Non-negotiables
- Do not store credentials in plain table fields.
- Do not embed credentials in URLs.
- Do not allow duplicate case creation for the same reminder.
- Do not omit the paired test app.
- Do not hardcode HTTP transport — use an interface.

## Allowed modernization
- Rename all "Kredinor" identifiers to "Collectia".
- Replace monolithic WS codeunit with separated concerns.
- Eliminate copy-pasted SOAP response checking.
- Move SOAP calls to background processing.
- Add telemetry for case creation outcomes.
- Add proper error handling with try/finally for logout.
- Replace raw string-concatenated SOAP with a structured message builder.
- Upgrade to modern runtime, application, and Cloud target.
- Add proper `DataClassification` to all fields.
- Add install and upgrade codeunits.
