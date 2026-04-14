# Test Specification

## Test app role
No test app exists. A rebuilt version must include a paired test app. The current architecture is untestable because HTTP calls are hardcoded. The rebuild must introduce an interface for the HTTP/SOAP transport so tests can use stubs.

## Required test structure
- Required domains mirrored in test app:
  - Setup
  - CaseCreation
  - DocumentUpload
  - ReminderIntegration
- Required helpers/stubs/spies:
  - SOAP transport stub (implements the HTTP transport interface with canned responses)
  - test helper for creating setup, customer, and issued reminder fixtures
- Required permission artifacts:
  - test permission set

## Acceptance criteria
| ID | Scenario | Given | When | Then |
|----|----------|-------|------|------|
| AC-SETUP-001 | Setup auto-creates on open | no setup record | page opens | record is created |
| AC-CASE-001 | Manual case creation succeeds | valid setup, valid reminder without case number | user confirms dialog | case number stored on reminder, sent flag set |
| AC-CASE-002 | Duplicate case blocked | reminder already has a case number | user tries to send again | error raised |
| AC-CASE-003 | Empty summons text blocked | user clears summons text in dialog | user confirms | error raised |
| AC-CASE-004 | User cancels dialog | reminder without case number | user cancels dialog | error raised, no changes |
| AC-CASE-005 | Danish VAT validation | customer with VAT number > 8 digits, Danish country | case creation attempted | error raised |
| AC-CASE-006 | Danish postal code validation | reminder with postal code > 4 digits, Danish country | case creation attempted | error raised |
| AC-CASE-007 | Foreign customer uses foreign payload | customer with non-Danish country code | case created | UDL type used, address mapped differently |
| AC-CASE-008 | Customer blocking when enabled | setup has block-customer enabled | case created successfully | customer Blocked = All |
| AC-CASE-009 | Customer not blocked when disabled | setup has block-customer disabled | case created successfully | customer not modified |
| AC-AUTO-001 | Automatic send on configured level | reminder level has send-to-Collectia enabled | reminder is issued | case created automatically |
| AC-AUTO-002 | No automatic send on unconfigured level | reminder level does not have send flag | reminder is issued | nothing happens |
| AC-DOC-001 | Sales invoice PDF uploaded | reminder line references a sales invoice | document upload runs | SOAP request contains base64 PDF and FAK type |
| AC-DOC-002 | Service invoice PDF uploaded | reminder line references a service invoice | document upload runs | SOAP request contains base64 PDF and FAK type |
| AC-DOC-003 | Non-invoice document metadata sent | reminder line references a payment | document upload runs | metadata-only request with BET type |
| AC-DOC-004 | Additional fees aggregated and sent | reminder has fee lines | document upload runs | single GEB request with summed amount |
| AC-DOC-005 | Document type mapping is correct | each BC document type | type is mapped | correct Collectia amount type code used |
| AC-LOG-001 | Activity log written when enabled | logging enabled in setup | case creation runs | activity log entry exists |
| AC-LOG-002 | No activity log when disabled | logging disabled in setup | case creation runs | no activity log entry |

## Mandatory negative tests
- duplicate case prevention
- empty summons text
- user cancel
- Danish VAT too long
- Danish postal code too long
- SOAP error response handling

## Minimum regression suite
- manual case creation happy path
- automatic case creation happy path
- duplicate prevention
- all validation rules
- document type mapping
- customer blocking toggle
- activity logging toggle
