# Solution Inventory

## Source Inventory Summary
| Object kind | Count | Notes |
|-------------|-------|-------|
| Table | 1 | Core operation log table |
| Pages | 4 | Operation list/card and scheduled task list/card |
| FactBox page | 1 | Payload viewer |
| Codeunits | 5 | Operation management, scheduling, error handling, JSON encoders/decoders, viewer management |
| Enums | 6 | HTTP method, operation status/result, JSON result, viewer direction, test endpoint |
| Interfaces | 1 | Request execution abstraction |
| Control add-ins | 1 | Browser-based payload viewer |
| Permission sets | 2 | Production and test |
| Test codeunits | 5 | Api operation, management, status/message utilities, encoders/decoders, comment merging |
| Test helper codeunits | 1 | Shared test fixture creation |
| Translation files | 1 | Danish translations |
| Web assets | 6 | Viewer scripts and stylesheet |

## Domain Breakdown
| Domain | Purpose | Included objects |
|--------|---------|------------------|
| ApiOperation | Persistent API operation tracking and orchestration | operation table, operation pages, operation management, task coordinator, error handler, status/result/method enums, request interface |
| Json | Primitive JSON conversion helpers | encoders, decoders, JSON result enum |
| JsonViewerFactBox | Human-readable request/response inspection | viewer management, viewer FactBox page, query direction enum, control add-in, scripts, stylesheet |
| ScheduledTasks | Administrative visibility into background tasks | scheduled task list and card pages |
| Permissions | User/test access packaging | ApiCore, ApiCoreTest |
| Test support | Regression and contract verification | test codeunits, helper, test endpoint enum |

## Object Inventory
| Object kind | Object name | Domain | Purpose | Required for rebuild | Depends on |
|-------------|-------------|--------|---------|----------------------|------------|
| Table | Api Operation | ApiOperation | Stores one API operation lifecycle record | Yes | operation enums, pages, management code |
| Enum | Api Method | ApiOperation | Standardized HTTP method classification | Yes | operation table and precondition logic |
| Enum | Api Operation Status | ApiOperation | Tracks operation lifecycle state | Yes | operation table, scheduling, error handling |
| Enum | Api Operation Result | ApiOperation | Tracks operation outcome classification | Yes | operation table, success/failure logic |
| Interface | IApiRequest | ApiOperation | Abstraction for request implementations | Yes | dependent integration codeunits |
| Codeunit | Api Operation Mgt | ApiOperation | Core validation, payload, HTTP-result, resend, classification logic | Yes | operation table, status/result enums |
| Codeunit | Api Operation Task Coordinator | ApiOperation | Scheduling, bulk scheduling, manual debug execution | Yes | operation table, error handler, BC scheduler |
| Codeunit | Api Operation Error Handler | ApiOperation | Default error capture for failed executions | Yes | operation table, status enum |
| Page | Api Operation List | ApiOperation | Technical list of all API operations | Yes | operation table, task coordinator |
| Page | Api Operation Card | ApiOperation | Detailed diagnostic view for a single API operation | Yes | operation table, viewer, operation management |
| Enum | Result | Json | Success/failure classification for JSON helper scenarios | Optional in v1 if no consumer, but present in source | JSON domain |
| Codeunit | Encoders | Json | Writes primitive values into JSON structures | Yes | dependent apps and tests |
| Codeunit | Decoders | Json | Reads primitive values from JSON structures and merges comments | Yes | dependent apps and tests |
| Codeunit | Universal Data Viewer Mgt | JsonViewerFactBox | Opens payload viewer with JSON/XML/text content | Yes | API log viewer page |
| Page | API Log Json Viewer | JsonViewerFactBox | Displays request/response payloads interactively | Yes | query direction enum, control add-in |
| Enum | Query Direction | JsonViewerFactBox | Chooses request vs. response payload display | Yes | viewer page |
| Control add-in | JsonViewer | JsonViewerFactBox | Rich in-client payload rendering | Yes | viewer page, web assets |
| Page | Scheduled Tasks List | ScheduledTasks | Administrative list of scheduled tasks | Yes | BC scheduled task table, task coordinator |
| Page | Scheduled Task Card | ScheduledTasks | Administrative detail page for a scheduled task | Yes | BC scheduled task table |
| Permission set | ApiCore | Permissions | Grants use of core technical objects | Yes | all production objects |
| Enum | Api Core Test Endpoint | Test support | Minimal endpoint selector used in tests | Yes for paired test app | tests |
| Codeunit | Test Api Operation | Test support | Verifies table- and precondition-level contract | Yes for paired test app | operation table, helper |
| Codeunit | Test Api Operation Mgt | Test support | Verifies payload, status, and HTTP outcome logic | Yes for paired test app | operation management |
| Codeunit | Test Api Operation Mgt2 | Test support | Verifies response classification and HTTP message mapping | Yes for paired test app | operation management |
| Codeunit | Test Add Comment | Test support | Verifies comment merge helper semantics | Yes for paired test app | decoders |
| Codeunit | Test Encoders and Decoders | Test support | Verifies primitive JSON round-tripping | Yes for paired test app | encoders, decoders |
| Codeunit | Test Helper | Test support | Creates valid baseline operation fixtures | Yes for paired test app | operation management, table |
| Permission set | ApiCoreTest | Permissions | Grants execution of test artifacts | Yes for paired test app | all test objects |

## Non-code Assets
| Asset | Purpose | Rebuild relevance |
|-------|---------|-------------------|
| Logo.png in each app | Branding | Low |
| Danish XLF translation file | Localization support | Medium |
| Viewer scripts and stylesheet | Payload visualization | High if rich viewer is preserved |
| VS Code launch/settings files | Development posture and analyzer posture | Medium |
| AGENTS.md in test app | workspace instruction artifact, not app behavior | None |