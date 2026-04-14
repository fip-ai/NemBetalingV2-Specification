# Integrations

## External contracts

### Generic outbound API request execution
- Purpose: Provide a reusable abstraction for outbound API calls made by dependent apps.
- Direction: outbound
- Dependency abstraction: request implementations are abstracted behind a request interface.

#### Operations
| Operation | Inputs | Expected outcome | Failure semantics |
|-----------|--------|------------------|-------------------|
| Send request | an Api Operation record containing endpoint, method, payload, URL, and handler context | external request is attempted and operation record is updated according to outcome | transport failure produces connection-error classification; non-2xx response produces service-failure classification |
| Accepts operation | an Api Operation record | implementation indicates whether it can handle the operation | unsupported operations should be declined by the implementation |

#### Payload meaning
| Element / payload concept | Meaning | Required / optional |
|---------------------------|---------|---------------------|
| Endpoint | logical target operation | Required before send |
| Request method | HTTP-style verb | Required before send |
| Request payload | body content for create/update style operations | Required for POST/PATCH/PUT, forbidden for GET/DELETE/HEAD/OPTIONS |
| Request URL | resolved external address | Optional in current infrastructure but strongly expected for diagnostics |
| Response payload | body returned by external system | Optional |
| Response status code | response classification signal | Optional until execution occurs |

#### Response meaning
| Response concept | Meaning | App reaction |
|------------------|---------|-------------|
| 2xx response | successful endpoint call | store response payload if present, mark result Success, mark status Completed |
| Non-2xx response | remote service returned failure | mark result WebService Call Failed, mark status Error |
| No successful send / connection failure | technical failure before valid success response | mark result Connection Error, mark status Error |

## Scheduled background execution
- Purpose: Allow asynchronous processing of API operations.
- Direction: internal platform integration
- Dependency abstraction: Business Central task scheduler plus runnable request/error handler codeunits.

#### Operations
| Operation | Inputs | Expected outcome | Failure semantics |
|-----------|--------|------------------|-------------------|
| Schedule one operation | operation record with request handler | background task is created, linked, and operation becomes Scheduled | returns failure if handler missing or task creation fails |
| Schedule all new operations | set of New operations | each eligible operation is scheduled | bulk run raises error when scheduling fails |
| Clear scheduled tasks | scheduled task records in relevant range | linked operations are reset to New and tasks cancelled | cancellation behavior relies on system platform services |

## Payload viewing integration
- Purpose: Provide rich interactive rendering of payload data in the client.
- Direction: internal UI integration
- Dependency abstraction: control add-in and helper codeunit.

#### Operations
| Operation | Inputs | Expected outcome | Failure semantics |
|-----------|--------|------------------|-------------------|
| Show JSON content | request and/or response text | viewer opens and displays content | if viewer not initialized yet, data waits until ready |
| Show XML content | request and/or response text | viewer opens and displays XML-like content through the universal viewer | malformed content may fall back to plain text behavior |
| Show arbitrary content | request and/or response text | viewer opens and displays detected content type | detection is best effort |

## Events and extension points relevant to integrations
- Request execution is intentionally abstracted through an interface so concrete integrations can be supplied by dependent apps.
- The operation record stores handler identifiers rather than embedding a single fixed transport implementation.

## Integration posture
ApiCore does not implement a specific external provider itself. It provides the contract and orchestration layer that other apps can plug into.