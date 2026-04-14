# Data Model

## Tables

### Collectia Setup
- Purpose: Singleton configuration record storing Collectia connection credentials, web-service endpoint, case-creation defaults, and behavioral switches.
- Persistence role: single-record application setup.
- Record lifecycle summary: auto-created when setup page is opened if missing.

#### Fields
| Field | Type | Required | Default | Editable | Business meaning | Validation / constraints |
|------|------|----------|---------|----------|------------------|--------------------------|
| Primary Key | Code up to 10 | System | blank | No | singleton anchor | clustered primary key |
| Customer No. | Code up to 20 | Yes | blank | Yes | the company's customer identifier at Collectia | used in all SOAP requests as client number |
| Block Customer | Boolean | No | false | Yes | whether to block the BC customer record when a case is created | blocks all transactions for the customer |
| WS Username | Text up to 30 | Yes | blank | Yes | web-service login username | **CRITICAL: must move to isolated storage in rebuild** |
| WS Password | Text up to 30 | Yes | blank | Yes | web-service login password | **CRITICAL: must move to isolated storage in rebuild** |
| WS Endpoint | Text up to 100 | Yes | blank | Yes | SOAP endpoint URL | base URL for all web-service calls |
| Summons Text | Text up to 100 | Conditionally | blank | Yes | default summons text for automatic case creation | required when automatic sending is enabled |
| Job URL | Text up to 100 | No | blank | Yes | URL template for opening a case in the Collectia web portal | used with format substitution for customer/credentials/job |
| Use Default Summons Text | Boolean | No | false | Yes | whether the default summons text is pre-filled in the manual input dialog | convenience setting |
| Begin Case in Default | Enum | No | INK | Yes | default case-start mode (collection vs reminder service) | pre-selects mode in manual dialog |
| Enable Log | Boolean | No | false | Yes | whether SOAP requests are logged to the activity log | troubleshooting switch |
| Course | Text up to 5 | No | blank | Yes | collection workflow/course code sent to Collectia | controls the processing path at Collectia |

#### Keys and uniqueness
- Primary key: Primary Key (singleton pattern).

#### Lifecycle rules
- Setup page auto-creates the record on first open.
- No delete allowed from UI.

## Table extensions

### Issued Reminder Header extension
- Base object: Issued Reminder Header (BC standard)
- Purpose: Track whether a reminder has been sent to Collectia and store the resulting case number.
- Added fields:

| Field | Type | Editable | Business meaning |
|------|------|----------|------------------|
| Send to Kredinor | Boolean | No (set by system) | flag indicating the reminder was transferred to Collectia |
| Kredinor Job No. | Code up to 20 | No (set by system) | the Collectia case number returned after successful creation |

### Reminder Level extension
- Base object: Reminder Level (BC standard)
- Purpose: Allow per-level configuration of automatic collection transfer.
- Added fields:

| Field | Type | Editable | Business meaning |
|------|------|----------|------------------|
| Send Reminder to Kredinor | Boolean | Yes | when enabled, issuing a reminder at this level automatically creates a Collectia case |

## Enums

### Begin Case in Default
- Purpose: Determines whether the case starts in collection mode or reminder-service mode.
- Extensible: Yes.
- Used by: setup table, input dialog.

| Value | Meaning | Business effect |
|-------|---------|-----------------|
| INK | Inkasso (debt collection) | case is created in collection workflow at Collectia |
| RYK | Rykkerservice (reminder service) | case is created in reminder-service workflow at Collectia |

## Rebuild notes for data model
- Credentials must move to isolated storage. They must not remain as plain text fields.
- The enum values currently lack captions. Rebuild must add proper Danish/English captions.
- Consider renaming all "Kredinor" identifiers to "Collectia" during rebuild for consistency with current business name.
- Consider adding `DataClassification` to all fields (missing in current source).
