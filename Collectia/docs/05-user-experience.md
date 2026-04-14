# User Experience

## Pages

### Collectia Setup
- Type: Card
- Purpose: Configure Collectia connection and case-creation behavior.
- Target user: finance administrator.
- Source entity: Collectia Setup table.

#### Layout contract
| Section | Fields / content | Purpose |
|---------|------------------|---------|
| General | customer number, block customer flag, default summons text flag, summons text, default case mode, course | case-creation behavior |
| Webservice setup | username, password, endpoint, job URL | connection credentials and endpoint |
| Log | enable log flag | troubleshooting |
| Version | hardcoded version label | informational |

#### Actions
| Action | Availability | Effect |
|--------|--------------|--------|
| Activity Log | Navigation action | opens activity log filtered to INCASSO context |

#### Conditional behavior
- Auto-creates setup record on first open.
- Insert and delete are disallowed.

### Collectia Input Dialog
- Type: StandardDialog
- Purpose: Collect summons text and case-start mode before manual case creation.
- Target user: accounts receivable clerk.

#### Layout contract
| Section | Fields / content | Purpose |
|---------|------------------|---------|
| Transfer confirmation | case-start mode (inkasso / rykkerservice) | user selects processing path |
| Summons text | multiline text field | user provides or confirms summons description |

#### Conditional behavior
- Pre-populates case mode from setup defaults.
- Pre-populates summons text if default-text flag is enabled in setup.

### Issued Reminder page extension
- Type: Page extension on Issued Reminder
- Purpose: Show collection fields and provide manual send/open/log actions.

#### Added fields
- Send to Collectia flag (read-only)
- Collectia Job No. (read-only)

#### Added actions
| Action | Availability | Effect |
|--------|--------------|--------|
| Send reminder to Collectia | Always available | triggers manual case creation flow |
| Open Collectia case | Only when a case number exists | opens Collectia web portal for the case |
| Activity Log | Always available | shows activity log entries for this reminder |

### Reminder Levels page extension
- Type: Page extension on Reminder Levels
- Purpose: Allow per-level configuration of automatic Collectia transfer.

#### Added fields
- Send Reminder to Collectia checkbox (editable)

## Navigation
- Setup is accessed from standard administration search.
- Case creation is triggered from the Issued Reminder page.
- Activity log is accessible from both setup and issued reminder pages.
