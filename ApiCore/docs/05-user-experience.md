# User Experience

## Pages

### Api Operation List
- Type: List
- Purpose: Provide a technical workbench for browsing, scheduling, debugging, and reviewing API operation history.
- Target user: Technical support, consultant, or developer.
- Source entity: Api Operation.

#### Layout contract
| Section | Fields / content | Purpose |
|---------|------------------|---------|
| Main repeater | entry number, description, status, associated record, endpoint, method, request time, execution time, response status code, result | Quick triage of operation state and outcome |

#### Actions
| Action | Availability | Effect |
|--------|--------------|--------|
| Schedule Task | On current row | Creates a background task for the selected operation |
| Schedule New Tasks | On page | Schedules all operations currently marked New |
| Manual Process / Debug Task | On current row | Runs the request handler immediately for troubleshooting |
| Scheduled Tasks | Navigation action | Opens the administrative list of background tasks |

#### Conditional behavior
- The associated-record display supports drill-down only when a linked record exists.
- The page is read-only and does not permit insert, modify, or delete.

#### Diagnostics / FactBoxes / technical aids
- This page is designed for operational visibility rather than business data entry.

### Api Operation Card
- Type: Card
- Purpose: Show full technical details of one API operation, including request/response payloads and handler identity.
- Target user: Technical support, consultant, or developer.
- Source entity: Api Operation.

#### Layout contract
| Section | Fields / content | Purpose |
|---------|------------------|---------|
| General | entry number, request time, associated record, status, description | Identity and traceability |
| Interface | request handler identifier and resolved name, error handler identifier and resolved name | Diagnose which implementation handled the operation |
| Request | endpoint, request URL, method, execution time | Understand what was attempted |
| Response | response status code, human-readable status message, result, raw response payload, payload size | Understand what came back and whether it succeeded |
| FactBoxes | links, notes, payload viewer | Extended technical diagnostics |

#### Actions
| Action | Availability | Effect |
|--------|--------------|--------|
| Send Request | Available from processing/promoted area | Resets prior success result and invokes the request handler again |

#### Conditional behavior
- Record drill-down is available only when an associated record exists.
- Handler names are resolved only when the referenced codeunit identifiers map to known objects.
- The page is read-only.

#### Diagnostics / FactBoxes / technical aids
- Embedded payload viewer supports side-by-side request/response inspection.
- Human-readable HTTP status text is shown in addition to the numeric code.

### API Log Json Viewer
- Type: CardPart FactBox
- Purpose: Render request and response payload content in a richer interactive viewer.
- Target user: Technical user diagnosing integrations.
- Source entity: Payload text supplied by parent page or helper.

#### Layout contract
| Section | Fields / content | Purpose |
|---------|------------------|---------|
| Payload direction selector | request vs response | Switch between outbound and inbound content |
| Viewer control | structured or text content | Human-readable payload inspection |

#### Actions
| Action | Availability | Effect |
|--------|--------------|--------|
| Request Payload | When currently showing response and comparison is meaningful | Switches viewer to request content |
| Response Payload | When currently showing request and comparison is meaningful | Switches viewer to response content |

#### Conditional behavior
- Payload direction controls are shown only when comparison mode is active.
- The viewer loads only after the control add-in signals readiness.
- The viewer supports both structured JSON mode and generic text mode.

#### Diagnostics / FactBoxes / technical aids
- Intended specifically as a technical diagnostics aid.
- Supports JSON, XML-like, and plain text payload inspection.

### Scheduled Tasks List
- Type: List
- Purpose: Show and manage scheduled background tasks related to API operations.
- Target user: Technical administrator or support user.
- Source entity: Business Central scheduled task system table.

#### Layout contract
| Section | Fields / content | Purpose |
|---------|------------------|---------|
| Main repeater | readiness, user name, run/failure handlers, company, earliest run time, record link, timeout, tenant id | Operational visibility into queued jobs |

#### Actions
| Action | Availability | Effect |
|--------|--------------|--------|
| Clear All Scheduled Tasks | On page | Cancels matching custom scheduled tasks and resets linked API operations |

### Scheduled Task Card
- Type: Card
- Purpose: Show detailed properties of one scheduled task.
- Target user: Technical administrator or support user.
- Source entity: Business Central scheduled task system table.

#### Layout contract
| Section | Fields / content | Purpose |
|---------|------------------|---------|
| General | identifiers, readiness, timing, run/failure handlers, linked record, timeout, tenant/auth delegation details | Deep technical inspection |
| User | user identifiers, language, formatting, time zone, application id | Execution-context diagnostics |

## Navigation
- Operation list opens operation card for detailed review.
- Operation list links to scheduled task list.
- Scheduled task list opens scheduled task card.
- Operation card allows drill-down into the originating business record when linked.

## UX posture
The app is intentionally technical and support-oriented. It favors transparency, traceability, and troubleshooting over end-user simplicity.