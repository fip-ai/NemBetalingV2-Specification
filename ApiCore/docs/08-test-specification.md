# Test Specification

## Test app role
The test app protects the technical contract of ApiCore. It proves that the shared infrastructure behaves predictably for dependent apps, especially around send preconditions, status/result classification, payload persistence, response interpretation, and JSON helper behavior.

## Required test structure
- Required domains mirrored in test app:
  - ApiOperation
  - Helpers
  - Permissions
- Required helpers/stubs/spies:
  - shared test helper for initializing a valid Api Operation fixture
- Required permission artifacts:
  - ApiCoreTest permission set

## Acceptance criteria
| ID | Scenario | Given | When | Then |
|----|----------|-------|------|------|
| AC-API-001 | Operation records are non-deletable | a persisted Api Operation record | deletion is attempted | the deletion is rejected |
| AC-API-002 | Missing endpoint blocks send | an operation without endpoint | send preconditions are evaluated | an explicit endpoint-required rejection is returned |
| AC-API-003 | POST requires payload | a POST operation without payload | send preconditions are evaluated | an explicit payload-required rejection is returned |
| AC-API-004 | GET must not contain payload | a GET operation with payload | send preconditions are evaluated | an explicit no-payload rejection is returned |
| AC-API-005 | Valid operation passes preconditions | an operation with endpoint, POST method, and payload | send preconditions are evaluated | no rejection is returned |
| AC-API-006 | Successful operations cannot be resent directly | an operation already marked successful | send preconditions are evaluated | a duplicate-success rejection is returned |
| AC-API-007 | Success predicate is false for non-success results | an operation with non-success result | success is evaluated | the result is false |
| AC-API-008 | Success predicate is true for success result | an operation marked Success | success is evaluated | the result is true |
| AC-API-009 | Connection failure classification is preserved | an operation pending execution | send failure is handled as technical failure | result becomes Connection Error and status becomes Error |
| AC-API-010 | Successful 2xx response completes the operation | an operation and a 2xx status with response body | success result is handled | status code is stored, payload is stored, result becomes Success, status becomes Completed |
| AC-API-011 | Non-2xx response becomes service failure | an operation and a non-2xx status | outcome is handled | response status code is stored, result becomes WebService Call Failed, status becomes Error |
| AC-API-012 | Request payload round-trips through storage | an operation and request payload text | payload is stored and then read | the same payload is returned |
| AC-API-013 | Response payload round-trips through storage | an operation and response payload text | payload is stored and then read | the same payload is returned |
| AC-API-014 | Status code 200 maps to OK | HTTP status code 200 | status message is resolved | the human-readable message is OK |
| AC-API-015 | Every status code from 0 to 511 returns a message | each status code in that range | status message is resolved | a non-empty message is returned |
| AC-API-016 | HTML response classification works | HTML content | response type is identified | type is HTML |
| AC-API-017 | JSON response classification works | JSON content | response type is identified | type is JSON |
| AC-API-018 | XML response classification works | XML content | response type is identified | type is XML |
| AC-API-019 | Plain text response classification works | plain text content | response type is identified | type is TEXT |
| AC-JSON-001 | Primitive boolean round-trip works | boolean true/false values | value is encoded then decoded | original value is preserved |
| AC-JSON-002 | Primitive date round-trip works | a valid or zero date | value is encoded then decoded | original value is preserved |
| AC-JSON-003 | Primitive datetime round-trip works | a valid or zero datetime | value is encoded then decoded | original value is preserved |
| AC-JSON-004 | Primitive decimal round-trip works | positive, negative, and zero decimal values | value is encoded then decoded | original value is preserved |
| AC-JSON-005 | Primitive integer round-trip works | positive, negative, and zero integer values | value is encoded then decoded | original value is preserved |
| AC-JSON-006 | Primitive text round-trip works | empty, normal, and special-character text | value is encoded then decoded | original value is preserved |
| AC-JSON-007 | Comment merging avoids duplicates | existing combined comments and a repeated comment | comment is added | stored text does not change |
| AC-JSON-008 | Comment merging adds a new unique comment | existing combined comments and a new comment | comment is added | new comment is appended with separator |
| AC-JSON-009 | Empty comment is ignored | any comment string and an empty new comment | comment is added | stored text does not change |

## Mandatory negative tests
- missing endpoint
- forbidden payload for body-less methods
- missing payload for body-bearing methods
- deletion prevention
- repeated-success resend prevention
- service failure classification
- connection failure classification

## Minimum regression suite
- operation creation creates durable non-deletable history rows
- precondition rules remain stable by request method
- 2xx success semantics remain stable
- non-2xx failure semantics remain stable
- request/response payload persistence remains stable
- response content classification remains stable
- primitive JSON encode/decode behavior remains stable
- comment-merging helper remains stable

## Rebuild note
A rebuilt ApiCore app is not done unless the paired test app enforces these acceptance criteria.