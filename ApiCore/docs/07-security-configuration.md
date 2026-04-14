# Security and Configuration

## Permission model

### ApiCore
- Intended role: technical user or dependent process needing access to API operation infrastructure.
- Access granted:
  - full use of the Api Operation table and its data
  - execution of operation management, task coordination, error handling, JSON helper, and viewer management code
  - access to operation pages, scheduled-task pages, and payload viewer page
- Dependencies: relies on platform/system objects outside the permission set’s direct control.

### ApiCoreTest
- Intended role: automated test execution context.
- Access granted:
  - execution of test codeunits and test helper codeunit
- Dependencies:
  - production app access via dependency on Api Core
  - Microsoft test framework dependencies

## Configuration model

This app does not define its own setup table in the inspected source.

Operational configuration is instead implicit in each Api Operation record:
- endpoint selection
n- request method
- request URL
- request handler
- error handler
- payload content

Dependent apps are expected to provide concrete configuration for real integrations.

## Environment and release posture
- Cloud / on-prem assumptions: test app explicitly targets Cloud; production app is effectively cloud-oriented but should be normalized to explicit target in rebuild.
- Telemetry expectations: no first-class telemetry implementation evidenced in current source.
- Resource exposure posture: currently permissive and developer-friendly; rebuild should treat this as a conscious decision rather than a default.
- Operational support posture: strong support/troubleshooting posture via operation logs, pages, payload viewer, and scheduled-task visibility.

## Sensitive data considerations
- Request and response payloads may contain sensitive integration data depending on dependent-app usage.
- The infrastructure stores payload bodies durably, so rebuilders should consider whether retention, masking, or role restriction needs strengthening even if the legacy app did not.

## Rebuild guidance
- Preserve the production permission set and paired test permission set.
- Consider hardening manifest metadata and exposure settings during rebuild.
- Consider adding explicit configuration concepts only if required by dependent apps, not as speculative expansion.