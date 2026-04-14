# Domain Behavior

## API operation lifecycle management

### Invariants
- Every persisted API operation has a sequential technical identifier.
- API operation history is intentionally non-deletable.
- A successful operation is treated as completed history and cannot be sent again until explicitly reset.
- Send preconditions depend on both endpoint selection and method/payload compatibility.

### Preconditions
- An operation must identify an endpoint before it can be sent.
- Payload-bearing methods require a request payload.
- Retrieval/delete/discovery methods must not carry a request payload.
- Scheduled or debug execution requires a request handler implementation.

### Validations and rejections
| Scenario | Rejection / prevention | Business reason |
|----------|------------------------|-----------------|
| Endpoint missing | Sending is blocked | The system must know what operation is being requested |
| GET/DELETE/HEAD/OPTIONS with payload | Sending is blocked | These request styles are treated as body-less operations in this infrastructure |
| POST/PATCH/PUT without payload | Sending is blocked | Update/create style requests require a body |
| Already successful operation is sent again | Sending is blocked until result is reset | Prevents accidental duplicate execution |
| User tries to delete operation log | Deletion is blocked | Preserves technical audit trail |
| Scheduling requested without request handler | Scheduling fails | No executable implementation exists |

### Core rules
- HTTP success is defined broadly as any 2xx status code.
- A transport/send failure is classified differently from a remote-service failure.
- Response payload is stored only when the response is considered successful and the response body is non-empty.
- When a request is explicitly reset for resend, the result classification is cleared before invoking the handler again.

### Calculations or transformations
- Stored request and response payloads are converted back to text using a best-effort encoding strategy: UTF-8 first, then Windows encoding, then unspecified fallback.
- Response status messages are produced from an internal lookup of well-known HTTP status codes.
- Response content can be classified as HTML, JSON, XML, or plain text based on content inspection.

### Result semantics
- Blank result means not yet evaluated or intentionally reset.
- Success means the endpoint returned a 2xx response and the operation reached completed state.
- Connection Error means the send attempt failed before receiving a valid success/failure response from the endpoint.
- WebService Call Failed means the endpoint responded, but with a non-2xx status.

### State-driven behavior
| State / classification | Expected behavior |
|------------------------|-------------------|
| New | Can be picked up for scheduling |
| Scheduled | Has an associated task identifier and is waiting for task execution |
| Error | Contains or should contain an error explanation and may be reviewed or retried |
| Completed | Represents successful finished history |
| Success result | Prevents direct resend without reset |

### Error semantics
- The default error handler captures the latest error text, stores it in the operation, marks the status as Error, and commits the update.
- Manual debug execution uses the same error-handling pathway when the request handler fails.

## Background scheduling behavior

### Invariants
- Bulk scheduling only targets operations currently marked New.
- Scheduled tasks are created a few seconds in the future, not immediately.
- Each scheduled operation receives a task identifier and transitions to Scheduled.

### Core rules
- If an operation has no explicit error handler, the built-in error handler is assigned during scheduling/debug use.
- Clearing scheduled tasks resets affected operations back to New and removes their scheduled-task link.
- Clearing tasks is limited to tasks within the relevant codeunit range and current company context.

## JSON encoding/decoding helper behavior

### Invariants
- Primitive encode/decode helpers preserve value identity for supported primitive types.
- Missing or null JSON values decode to neutral defaults rather than raising errors.

### Validations and rejections
| Scenario | Rejection / prevention | Business reason |
|----------|------------------------|-----------------|
| Comment already present in combined comment text | Duplicate addition prevented | Avoids repetitive diagnostics/comments |
| New comment empty | No change made | Avoids meaningless separators |
| Combined comment would exceed destination capacity | No additional comment appended | Prevents overflow |

### Core rules
- Boolean, date, datetime, decimal, integer, and text values are round-tripped through JSON helpers.
- Decoding a missing or null value returns the type’s empty default.
- Comment aggregation keeps unique comments in insertion order, separated by a caller-supplied delimiter.

## Payload viewing behavior

### Invariants
- Technical users can inspect request and response payloads side by side through the card page’s FactBox.
- Viewer logic supports JSON, XML, and arbitrary text content.

### Core rules
- The viewer defaults to response display when initialized.
- If request content is available, the viewer supports toggling between request and response.
- XML and non-JSON text are still displayable through the universal viewer pathway.
- Content type detection is lightweight and intended for display behavior, not strict validation.
