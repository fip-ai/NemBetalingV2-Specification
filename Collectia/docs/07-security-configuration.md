# Security and Configuration

## Permission model

### Current state
No permission set exists. This is a blocker.

### Rebuild requirement
Create a permission set granting:
- setup table data access
- execution of WS codeunit and subscriber codeunit
- access to setup and input pages
- modification access to issued reminder header extension fields

## Configuration model

### Collectia Setup
- Purpose: single-record configuration for the entire integration.
- Required settings:
  - Customer No. (Collectia client identifier)
  - WS Username
  - WS Password
  - WS Endpoint
- Optional settings:
  - Block Customer (default: false)
  - Use Default Summons Text (default: false)
  - Summons Text (default for automatic sending)
  - Begin Case in Default (INK or RYK)
  - Course
  - Job URL
  - Enable Log (default: false)
- Consequences of missing config: SOAP calls will fail; no graceful pre-check exists.

## Sensitive data considerations

### Critical security issues in current implementation
1. **WS Username and WS Password are stored as plain Text fields.** This is a fundamental security violation. They must be moved to isolated storage or Azure Key Vault.
2. **Credentials are displayed unmasked on the setup page.** The password field must use `ExtendedDatatype = Masked`.
3. **Credentials are embedded in a browser URL** via the "Open case" HYPERLINK action. This exposes them in browser history, proxy logs, and potentially network traffic.

### Rebuild requirements
- Store credentials in company-scoped isolated storage.
- Display masked placeholders on the setup page.
- Remove credentials from URL construction entirely. If the Collectia portal requires authentication, handle it differently.
- Consider using `SecretText` type for credential handling.

## Environment and release posture
- Cloud / on-prem: no explicit Cloud target; rebuild must target Cloud.
- Telemetry: none; rebuild should add telemetry for case creation success/failure.
- Resource exposure: `showMyCode: false`; rebuild should reconsider based on customer needs.
- Operational support: activity log exists but only when manually enabled.
