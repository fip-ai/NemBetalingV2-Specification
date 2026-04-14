# Process Flows

## Create an API operation record

### Trigger
A dependent app decides that an outbound API interaction must be recorded and potentially executed.

### Main flow
1. Create a new operation record.
2. Assign the next sequential technical entry number.
3. Stamp the creation timestamp.
4. Populate descriptive context, endpoint, method, associated business record, handler references, and payload as applicable.
5. Persist the record.

### Alternate flows
- If the caller deliberately inserts a partial record, the record may exist in an incomplete state, but it will not pass send preconditions.

### Failure flow
- If required values are not supplied later, send validation blocks execution.

### State transitions
| From | To | Trigger | Conditions |
|------|----|---------|------------|
| Blank | New | Caller initializes a valid pending operation | Dependent app chooses to mark as new |

## Validate send preconditions

### Trigger
A caller attempts to send or re-send an API operation.

### Main flow
1. Confirm that an endpoint is selected.
2. Inspect the request method.
3. If the method is a body-less method, reject any existing payload.
4. If the method is a body-bearing method, require a payload.
5. Reject the operation if it already succeeded previously.
6. Allow execution only if no rule is violated.

### Alternate flows
- If no rule is violated, validation returns successfully without side effects.

### Failure flow
1. Identify the first violated precondition.
2. Produce a user-readable error message describing the rule breach.
3. Abort send processing.

### Retry / resend semantics
- Previously successful operations must first be reset before resend.

## Execute an API operation and process outcome

### Trigger
A request handler sends the external request and returns success/failure information to the management logic.

### Main flow
1. Record the HTTP-style status code.
2. If the status code is in the 2xx range, store the response payload when present.
3. Mark the result as Success.
4. Mark the status as Completed.

### Alternate flows
- If response content is empty but status is successful, the operation still completes successfully.

### Failure flow
1. If the external send fails before a usable success response, classify as Connection Error and mark status Error.
2. If the external service responds with a non-2xx status, classify as WebService Call Failed and mark status Error.
3. Do not store a normal response payload for non-success statuses in the built-in success pathway.

### State transitions
| From | To | Trigger | Conditions |
|------|----|---------|------------|
| New or in-process pending state | Completed | Successful 2xx response | Handler returns success outcome |
| New or in-process pending state | Error | Connection failure | Send attempt fails |
| New or in-process pending state | Error | Non-2xx response | Endpoint responds with failure status |

## Reset and resend a successful operation

### Trigger
A user invokes the resend action from the operation card.

### Main flow
1. Clear the prior result classification.
2. Persist the reset.
3. Invoke the request handler again using the same operation record.

### Alternate flows
- The handler may fully reuse the original payload and operation metadata.

### Failure flow
- Any downstream execution failure is handled by the normal request/error pathways.

## Schedule a single operation for background processing

### Trigger
A user schedules the current operation from the list page.

### Main flow
1. Verify that a request handler exists.
2. If no explicit error handler is set, assign the built-in error handler.
3. Create a scheduled task a few seconds in the future for the current company.
4. Store the scheduled task identifier on the operation.
5. Change operation status to Scheduled.
6. Persist and commit the operation update.

### Alternate flows
- A valid custom error handler can be preserved if already assigned.

### Failure flow
1. If no request handler exists, scheduling returns failure.
2. If scheduled task creation returns no identifier, scheduling returns failure and may show a user message in interactive sessions.

### State transitions
| From | To | Trigger | Conditions |
|------|----|---------|------------|
| New | Scheduled | Task created successfully | Request handler exists |

### Scheduling / background execution
- Tasks are created with a slight delay rather than immediate execution.
- Task scheduling is company-specific.

## Schedule all new operations

### Trigger
A user invokes bulk scheduling from the operation list page.

### Main flow
1. Find all operations with status New.
2. Attempt to schedule each one.
3. Leave each successfully scheduled operation in Scheduled state.

### Failure flow
1. If any operation fails to schedule, raise an error indicating task scheduling failure.
2. The exact partial-progress semantics are not fully specified by source comments, so rebuilders should preserve fail-fast visible error behavior while maintaining data integrity.

## Debug or manually process one operation

### Trigger
A user invokes manual processing/debug from the operation list page.

### Main flow
1. Clear the current stored error message.
2. Run the request handler directly in the foreground.
3. If the handler succeeds, return success to the caller.

### Failure flow
1. If the request handler fails, reload the operation from storage.
2. Ensure an error handler is available, defaulting to the built-in one if necessary.
3. Run the error handler for the operation.
4. Return failure to the caller.

## Clear scheduled tasks

### Trigger
A user invokes the scheduled-task cleanup action from the scheduled task list page.

### Main flow
1. Enumerate scheduled tasks for the current company within the relevant custom codeunit range.
2. For each matching scheduled task, find linked operations by scheduled task identifier.
3. Reset each linked operation to New.
4. Clear its scheduled task identifier.
5. Persist the change.
6. Cancel the underlying system scheduled task.

### Failure flow
- No explicit user-facing failure strategy is documented beyond the underlying task cancellation behavior.

## View payloads and diagnostics

### Trigger
A technical user opens the operation card or uses the viewer management helper.

### Main flow
1. Convert stored request and response payload blobs into text.
2. Feed the content into the payload viewer.
3. Default the viewer to response mode.
4. Allow the user to toggle to request mode if both are available.
5. If associated business record information exists, allow drill-down into that record.

### Alternate flows
- Non-JSON content may still be shown as raw text.
- XML content may be treated as displayable structured content through the universal viewer path.
