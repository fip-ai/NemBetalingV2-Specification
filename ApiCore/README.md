# ApiCore Specification

## Purpose

ApiCore is the shared infrastructure app in NemBetalingV2 for executing, tracking, and troubleshooting API operations inside Business Central. It provides:

- a persistent `Api Operation` model
- orchestration codeunits for execution and scheduling
- JSON encoding/decoding helpers
- pages for inspection and troubleshooting
- enums/interfaces that standardize request handling
- a dedicated test app validating the behavior

## Scope

This specification covers:

- `ApiCore`
- `ApiCore.Test`

## Main responsibilities

1. Represent an API operation as persisted business data.
2. Execute API requests through a managed workflow.
3. Coordinate scheduled/background processing.
4. Capture state, result, and error information for diagnostics.
5. Provide reusable JSON utilities.
6. Support verification through a separate automated test app.

## Source structure

### ApiCore
- `src/ApiOperation/` - operation model, management, coordination, pages, enums, interface
- `src/Json/` - encoders/decoders and JSON result enum
- `src/JsonViewerFactBox/` - generic data viewer helper
- `src/ScheduledTasks/` - scheduled task pages
- `src/Permissions/` - permission set

### ApiCore.Test
- `src/ApiOperation/` - tests around operation behavior and endpoints
- `src/Helpers/` - test helpers
- `src/Permissions/` - permission set

## Intended architectural role

ApiCore is a foundational technical app rather than an end-user business app. Other NemBetalingV2 apps are expected to depend on it for consistent request execution, result handling, and diagnostics.
