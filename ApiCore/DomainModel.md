# Domain Model

## Core entity: Api Operation

`Api Operation` is the central persisted record representing one API interaction lifecycle.

## Responsibilities of the entity

The table acts as the audit and execution anchor for:

- request intent
- operation type/category
- payload data
- execution status
- response/result data
- error details
- timestamps and traceability

## Supporting concepts

### Management codeunits
The `Api Operation Mgt` codeunit owns the main business logic for creating, updating, and progressing operations.

### Coordination codeunits
Coordinator codeunits organize higher-level flows, likely including deferred execution and scheduled processing.

### Interface-based execution
The app defines at least one interface for operation handling, allowing app-specific implementations to plug into the common orchestration model.

### Enums
Enums standardize:

- operation status/state
- operation type/category
- JSON parse/serialization outcomes

## UI model

Pages expose the operation records for debugging and administrative review. This suggests the table is intentionally user-inspectable by technical/support users.

## JSON support model

Json helpers provide a reusable abstraction layer so callers can:

- encode Business Central data into JSON
- decode incoming JSON
- interpret parsing/validation outcomes consistently

## Test model

`ApiCore.Test` mirrors the operational model with tests that validate management behavior and expected edge cases in operation processing.
