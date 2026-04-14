# Data Model

## Tables

### Api Operation
- Purpose: Represents one outbound API interaction attempt or completed interaction.
- Persistence role: Durable technical log, orchestration anchor, retry anchor, and troubleshooting record.
- Record lifecycle summary: Created when an API operation is initiated, enriched before execution, updated during scheduling/execution/error handling, and intentionally protected from deletion.

#### Fields
| Field | Type | Required | Default | Editable | Business meaning | Validation / constraints |
|------|------|----------|---------|----------|------------------|--------------------------|
| Entry No. | Integer | Yes | Next sequential number on create | No | Unique technical identifier for the operation | Assigned automatically at creation time |
| Endpoint | Text up to 50 | Conditionally required | Blank | Yes | Logical operation or endpoint name being targeted | Must be populated before sending |
| Request Method | Enum | Conditionally required | Blank | Yes | HTTP-style operation method | Payload rules depend on method |
| Request Payload | Large text/blob | Required for payload-bearing methods, forbidden for methods that do not accept body content | Empty | Indirectly editable | Outbound request body | Must be absent for retrieval/deletion-style methods and present for create/update-style methods |
| Requested At | DateTime | Yes | Current date-time on create | No | When the operation record was created | System-assigned |
| Associated Record Id | Record reference | No | Blank | No | Link to the business record that caused the operation | Used for drill-down and traceability |
| Api Operation Status | Enum | Yes | Blank unless explicitly set before insert | Yes | Lifecycle state of the operation | Updated by scheduling, success, and error processes |
| Description | Text up to 100 | No | Blank | Yes | Human-readable technical description | Commonly used to describe the intended operation |
| Request Url | Text up to 2048 | No | Blank | Yes | Fully resolved target URL | Optional but critical for diagnostics |
| Request Handler Codeunit | Integer reference to runnable handler | Required for scheduled/manual execution | Blank | Yes | Technical handler responsible for performing the request | Scheduling fails if missing |
| Error Handler Codeunit | Integer reference to runnable handler | Optional | Defaults to built-in error handler when needed | Yes | Technical handler for failed executions | Defaulted automatically when absent |
| Scheduled Task Id | GUID | No | Blank | Yes | Link to background task created for execution | Populated on scheduling, cleared when tasks are cancelled/reset |
| Line Style | Text up to 20 | No | Blank | Yes | Visual styling support for list presentation | No strong behavioral evidence in source |
| Error Message | Text up to 2048 | No | Blank | Yes | Last captured failure message | Populated by error handling |
| Executed At | DateTime | No | Blank | Yes | When execution actually took place | Present for audit/diagnostics |
| Response Payload | Large text/blob | No | Empty | Indirectly editable | Returned payload from the external endpoint | Stored only for successful 2xx outcomes when non-empty content exists |
| Response Status Code | Integer | No | 0 or blank semantics | Yes | HTTP-style response status | Used for result interpretation and human-readable message mapping |
| Api Operation Result | Enum | Yes | Blank | Yes | High-level result classification | Driven by execution outcome |

#### Relationships
| Field / concept | Related entity | Relationship type | Meaning |
|-----------------|----------------|-------------------|---------|
| Associated Record Id | Any BC record | loose traceability link | Connects the technical operation to the originating business record |
| Request Handler Codeunit | Request implementation codeunit | execution dependency | Defines which component can execute the operation |
| Error Handler Codeunit | Error handler codeunit | failure dependency | Defines which component records error state |
| Scheduled Task Id | Scheduled Task system record | background execution link | Connects the operation to the queued background job |

#### Keys and uniqueness
- Primary key: Entry No.
- Additional keys: none evidenced.
- Uniqueness semantics: one operation record per sequential entry number; no evidence of duplicate prevention by business content.

#### Computed or derived values
- Request payload text: derived by decoding stored blob content using several encoding attempts.
- Response payload text: derived by decoding stored blob content using several encoding attempts.
- Response status message: derived from response status code through an internal message map.

#### Lifecycle rules
- On create: assign next sequential entry number, stamp creation date-time.
- On update: no generic table-wide update rule evidenced.
- On delete: deletion is blocked with an explicit error. The record is intended to be immutable as historical evidence.
- On rename: no rename-specific rule evidenced.

## Enums and options

### Api Method
- Purpose: Classifies the request verb supported by the infrastructure.
- Extensible: Yes.
- Used by: Api Operation request method.

| Value | Meaning | Business effect |
|-------|---------|-----------------|
| Blank | Unspecified method | Operation is incomplete and not ready for execution |
| GET | Retrieval request | Request payload must not be present |
| POST | Create/submit style request | Request payload is required |
| PATCH | Partial update request | Request payload is required |
| PUT | Full replacement/update request | Request payload is required |
| DELETE | Delete request | Request payload must not be present |
| HEAD | Header-only request | Request payload must not be present |
| OPTIONS | Capability/discovery request | Request payload must not be present |

### Api Operation Status
- Purpose: Lifecycle state of an API operation.
- Extensible: Yes.
- Used by: Api Operation status field, scheduling and error flows.

| Value | Meaning | Business effect |
|-------|---------|-----------------|
| Blank | No lifecycle state assigned yet | Transitional or incomplete record |
| New | Ready to be scheduled or processed | Eligible for bulk scheduling |
| Scheduled | Background task created | Awaiting task execution |
| On Hold | Explicitly paused | Not automatically progressed by current source |
| In Progress | Execution underway | Reserved for active processing semantics |
| Error | Operation failed | Requires diagnostics or retry |
| Cancelled | Operation intentionally abandoned | Not processed further |
| Completed | Operation finished successfully | Eligible for history review, not for immediate re-send without reset |

### Api Operation Result
- Purpose: High-level outcome classification.
- Extensible: Yes.
- Used by: Api Operation result field, success checks, resend guard.

| Value | Meaning | Business effect |
|-------|---------|-----------------|
| Blank | No outcome yet | Still pending or reset for resend |
| Success | Request completed successfully | Prevents direct resend until reset |
| Connection Error | Technical send failure before successful response | Marks operation as failed |
| WebService Call Failed | Remote endpoint responded with non-success status | Marks operation as failed |

### Json Result
- Purpose: Generic success/failure classification in JSON helper domain.
- Extensible: No.
- Used by: No direct runtime consumer evidenced in current source; kept as shared utility classification.

| Value | Meaning | Business effect |
|-------|---------|-----------------|
| Blank | No result assigned | Neutral/unused state |
| Failure | JSON-related operation failed | Utility classification |
| Success | JSON-related operation succeeded | Utility classification |

### Query Direction
- Purpose: Chooses which payload stream the viewer displays.
- Extensible: No.
- Used by: API log viewer page.

| Value | Meaning | Business effect |
|-------|---------|-----------------|
| Blank | No direction chosen yet | Initial neutral state |
| Request | Show outbound payload | Enables comparison view of request |
| Response | Show inbound payload | Enables comparison view of response |

### Api Core Test Endpoint
- Purpose: Minimal endpoint catalog for tests.
- Extensible: not specified in a meaningful way for production use.
- Used by: Test fixtures and precondition tests.

| Value | Meaning | Business effect |
|-------|---------|-----------------|
| Blank | No endpoint selected | Used to test invalid/incomplete operations |
| Test | Placeholder endpoint | Used to create valid test operations |

## Extensions

This app does not define table extensions or enum extensions in the inspected source.
