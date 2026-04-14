# Workflows

## 1. Operation creation

A caller creates or requests an `Api Operation` through management code.

Expected steps:
1. Initialize operation metadata.
2. Persist the operation record.
3. Mark initial state.
4. Prepare payload/context for execution.

## 2. Operation execution

Execution is managed centrally rather than embedded in pages.

Expected steps:
1. Resolve the correct handler/implementation.
2. Execute request logic.
3. Capture success/failure outcome.
4. Persist result payload and diagnostics.
5. Transition operation state.

## 3. Scheduled/background processing

The presence of scheduled task artifacts indicates that some operations can be processed asynchronously.

Expected steps:
1. Identify queued or pending operations.
2. Dispatch through coordinator logic.
3. Execute in background context.
4. Update persisted state and diagnostics.

## 4. Troubleshooting and review

Technical users can inspect operations through pages/factboxes.

Typical review flow:
1. Open operation list/card.
2. Inspect payload/result JSON.
3. Review state and error fields.
4. Correlate with scheduled-task behavior if relevant.

## 5. Automated verification

The test app validates the workflow through AL tests.

Expected coverage:
- creation behavior
- state transitions
- management codeunit logic
- error handling
- integration boundaries around handlers/utilities
