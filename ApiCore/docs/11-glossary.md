# Glossary

| Term | Meaning | Notes for rebuild |
|------|---------|-------------------|
| Api Operation | One tracked outbound API interaction attempt/result | Core technical entity of the app |
| Endpoint | Logical name of the requested external operation | Not necessarily the full URL |
| Request handler | The executable component that knows how to perform the API request | Abstract behind an interface or equivalent modern contract |
| Error handler | The executable component that records failure state for an operation | Default implementation must exist |
| Operation status | Lifecycle state of the operation | Separate from result classification |
| Operation result | Outcome classification of the operation | Distinguishes success, connection failure, and remote failure |
| Scheduled task | Background job created by the BC platform | Linked back to Api Operation through a task identifier |
| Request payload | Stored outbound body content | Required only for body-bearing methods |
| Response payload | Stored inbound content from the external endpoint | Usually stored on successful responses |
| Payload viewer | Technical UI for inspecting request and response content | Important support feature |
| Query direction | Whether the viewer shows request or response | Viewer-only concept |
| Connection Error | Technical failure to send/complete the request | Different from remote business/service failure |
| WebService Call Failed | Remote endpoint responded with a non-success status | Must remain a distinct result class |
| Clean-room rebuild | Re-implementation from specification without consulting original source | Central project objective |
| Paired test app | Separate Business Central test extension validating the production app contract | Not optional in this reconstruction strategy |