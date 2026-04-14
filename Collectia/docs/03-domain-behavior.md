# Domain Behavior

## Case creation from issued reminders

### Invariants
- A reminder can only be sent to Collectia once. If it already has a case number, re-sending is blocked.
- Summons text is mandatory for every case creation.
- The customer's country determines whether the domestic or foreign payload template is used.

### Preconditions
- Setup record must exist with valid credentials and endpoint.
- Summons text must be provided (either manually or from setup defaults).
- The issued reminder must not already have a Collectia case number.

### Validations and rejections
| Scenario | Rejection / prevention | Business reason |
|----------|------------------------|-----------------|
| Reminder already has a case number | error raised immediately | prevents duplicate case creation at Collectia |
| User cancels the manual input dialog | error raised | action was deliberately aborted |
| Summons text is empty | error raised | Collectia requires a summons description |
| Danish customer with VAT number longer than 8 digits | error raised | Danish CVR must be max 8 digits |
| Danish customer with postal code longer than 4 digits | error raised | Danish postal codes are max 4 digits |

### Core rules — manual creation
1. User opens an issued reminder and triggers the send action.
2. An input dialog appears, pre-populated based on setup defaults.
3. User confirms or adjusts the summons text and case-start mode.
4. The app authenticates with Collectia via SOAP login and obtains a session token.
5. A case-creation SOAP request is built using customer, reminder, and setup data.
6. The request is sent and the response is parsed.
7. If successful, the returned case number is stored on the issued reminder header.
8. If customer blocking is enabled, the customer is blocked for all transactions.
9. Invoice copies and document metadata are uploaded as supporting documents.

### Core rules — automatic creation
1. When a reminder is issued in BC, an event subscriber fires.
2. The subscriber checks whether the reminder level is configured for automatic Collectia transfer.
3. If yes, the same case-creation logic runs automatically using setup defaults instead of dialog input.

### Result semantics
- Success: case number stored on reminder, customer optionally blocked, documents uploaded.
- Failure: SOAP error text surfaced as BC error. No partial state changes are persisted on failure before the case number is received.

### State-driven behavior
| State / classification | Expected behavior |
|------------------------|-------------------|
| Reminder without case number | eligible for sending |
| Reminder with case number | blocked from resending; "Open case" action becomes available |

### Error semantics
- SOAP login errors are surfaced as BC errors with the remote error text.
- Case-creation errors are surfaced as BC errors with the remote error text.
- A case number of "0" is treated as failure and the remote error text is shown.
- Document-upload errors (W or E return types) are surfaced as BC errors.

## Document upload to Collectia cases

### Invariants
- Documents are uploaded only after a case is successfully created.
- Each reminder line is processed individually.

### Core rules
1. After case creation, the app iterates all reminder lines of type Customer Ledger Entry.
2. For each line that is a sales invoice, a PDF is generated from the standard report and uploaded with invoice metadata.
3. For each line that is a service invoice, a PDF is generated from the service invoice report and uploaded.
4. For lines that are not invoices (payments, credit memos, finance charge memos, reminders, refunds), only metadata is sent without a PDF attachment.
5. The document type is mapped to a Collectia amount-type code.
6. Additional fee lines are aggregated and sent as a single fee document.
7. Negative amounts are converted to positive for the fee/credit submissions.

### Document-type mapping
| BC document type | Collectia amount type |
|------------------|-----------------------|
| Invoice | FAK |
| Payment | BET |
| Credit Memo | KRE |
| Finance Charge Memo | RNT |
| Reminder | GEB |
| Refund | KRE |
| Unspecified with negative amount | KRE |
| Unspecified with positive amount | FAK |
| Additional fee | GEB |

## Authentication lifecycle

### Core rules
1. Login obtains a session token using customer number, username, and password.
2. The token is used for all subsequent requests in the same workflow.
3. A logout procedure exists but is not evidenced as being called automatically after case creation.

### Rebuild note
The rebuild should ensure logout is always called after a workflow completes, even on failure. Consider a try/finally pattern.

## Customer blocking

### Core rules
- If the setup flag is enabled, the customer record is blocked for all transactions immediately after successful case creation.
- If the flag is disabled, no customer modification occurs.

## Activity logging

### Core rules
- When logging is enabled in setup, each SOAP request body is written to the BC Activity Log with context "INCASSO".
- A commit is issued after each log entry.
- The issued reminder page extension provides access to the activity log filtered to the current record.
