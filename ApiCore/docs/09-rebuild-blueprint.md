# Rebuild Blueprint

## App manifest blueprint
- App name: Api Core
- Test app name: Api Core Test
- App role: shared foundation app for outbound API operations and technical diagnostics
- Dependencies:
  - production app: base/system Business Central platform only in current source
  - test app: Api Core plus Microsoft test libraries
- Runtime/target posture:
  - runtime equivalent to BC runtime 16 generation
  - explicit Cloud target recommended for rebuilt production app
  - paired test app required

## Reconstruction order
1. Build production app manifests, folder structure, and core enums/interface.
2. Build Api Operation table and production permission set.
3. Build Api Operation management, error handling, and scheduling codeunits.
4. Build operation list/card pages.
5. Build payload viewer domain: viewer enum, FactBox page, control add-in wrapper, viewer management codeunit.
6. Build scheduled task list/card pages.
7. Build translation and manifest posture.
8. Build paired test app, helper, test permission set, and full acceptance suite.

## Object blueprint by domain

### ApiOperation
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Enum | Api Method | HTTP-style method classification with body-required/body-forbidden semantics |
| Enum | Api Operation Status | lifecycle state model including New, Scheduled, Error, Completed |
| Enum | Api Operation Result | high-level result classification including Success, Connection Error, WebService Call Failed |
| Interface | Request execution abstraction | one operation-sending contract and one acceptance contract |
| Table | Api Operation | durable operation record with request/response payloads, status/result, handlers, task link, associated record traceability, non-deletable history |
| Codeunit | Api Operation Mgt | send precondition checks, status/result updates, request/response payload persistence, response classification, resend reset |
| Codeunit | Api Operation Error Handler | store latest error message and mark operation Error |
| Codeunit | Api Operation Task Coordinator | single scheduling, bulk scheduling, manual debug execution, task cleanup |
| Page | Api Operation List | read-only list plus scheduling/debug/navigation actions |
| Page | Api Operation Card | detailed read-only diagnostics view plus resend action and embedded payload viewer |

### Json
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Codeunit | Encoders | primitive JSON writing helpers for boolean, date, datetime, decimal, integer, and text |
| Codeunit | Decoders | primitive JSON reading helpers with neutral defaults plus unique comment-merging helper |
| Enum | Json result classification | optional shared utility enum if needed by architecture |

### JsonViewerFactBox
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Enum | Query Direction | request/response display mode |
| Page | API Log Json Viewer | technical payload viewing FactBox with request/response toggle |
| Codeunit | Universal Data Viewer Mgt | convenience entry points for showing JSON/XML/text content |
| Control add-in wrapper | JsonViewer | rich viewer host abstraction; actual UI may be modernized if behavior is preserved |

### ScheduledTasks
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Page | Scheduled Tasks List | administrative view of queued tasks plus cleanup action |
| Page | Scheduled Task Card | detailed inspection page for one scheduled task |

### Permissions
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Permission set | ApiCore | grants use of production infrastructure |
| Permission set | ApiCoreTest | grants execution of test artifacts |

### Test app
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Enum | Test endpoint enum | minimal endpoint selector for valid/invalid fixture states |
| Codeunit | Test Helper | creates a valid baseline Api Operation fixture |
| Test codeunits | Contract tests | enforce acceptance criteria from the test specification |

## Mandatory behaviors
- Operation records are durable and non-deletable.
- Payload requirements vary by request method.
- Success means any 2xx status code.
- Transport failure and remote-service failure are distinct result classes.
- Successful resend requires explicit reset of the result before invoking the handler again.
- Scheduling assigns default error handling when none is provided.
- Bulk scheduling only targets New operations.
- Task cleanup resets linked operations to New and clears task identifiers.
- Payloads can be read back as text using tolerant decoding.
- Technical users can inspect payloads and linked business records from the UI.

## Mandatory tests for rebuilt solution
- Implement all acceptance criteria listed in `08-test-specification.md`.
- Maintain one positive and one negative test for each core precondition rule.
- Validate payload round-trip behavior.
- Validate response-type classification behavior.
- Validate JSON primitive round-trip behavior.
- Validate comment-merging duplicate suppression.

## Non-negotiables
- Do not make operation history deletable.
- Do not collapse Connection Error and WebService Call Failed into one generic failure state.
- Do not remove the request-method-specific payload rules.
- Do not omit the paired test app.
- Do not remove the technical diagnostic UI without replacing its troubleshooting value.

## Allowed modernization
- Strengthen manifest metadata and target posture.
- Replace numeric codeunit references with a cleaner architectural indirection if the same execution contract is preserved.
- Improve separation of concerns and testability.
- Replace the exact viewer implementation so long as request/response inspection capability remains.
- Add telemetry and stronger security posture if behavior is preserved.