# Process Flows

## Manual case creation

### Trigger
User opens an issued reminder and clicks "Send reminder to Collectia".

### Main flow
1. Verify the reminder does not already have a Collectia case number.
2. Open the input dialog, pre-populated from setup defaults.
3. User confirms summons text and case-start mode.
4. Authenticate with Collectia (SOAP login) and obtain session token.
5. Determine customer type: domestic (Denmark) or foreign, based on country/region code.
6. For domestic: validate VAT number is max 8 digits and postal code is max 4 digits.
7. Build the case-creation SOAP request with customer, reminder, and setup data.
8. Log the request if logging is enabled.
9. Send the request to Collectia.
10. Parse the response: extract case number.
11. Store case number and sent flag on the issued reminder header.
12. If customer blocking is enabled, block the customer for all transactions.
13. Upload invoice copies and document metadata for all reminder lines.

### Failure flow
1. If reminder already has a case number: error, stop.
2. If user cancels dialog: error, stop.
3. If summons text is empty: error, stop.
4. If domestic validation fails: error, stop.
5. If SOAP login fails: error with remote message, stop.
6. If case creation returns error or case number "0": error with remote message, stop.
7. If document upload returns error: error with remote message, stop.

### State transitions
| From | To | Trigger | Conditions |
|------|----|---------|------------|
| Reminder without case number | Reminder with case number + sent flag | successful case creation | all validations pass and Collectia returns a case number |

## Automatic case creation on reminder issue

### Trigger
BC issues a reminder (standard Reminder-Issue codeunit fires OnAfterIssueReminder event).

### Main flow
1. Look up the issued reminder by the returned number.
2. Find the reminder level configuration.
3. Check if automatic send is enabled for that level.
4. If yes, call the same case-creation logic with automatic mode (uses setup defaults, no dialog).

### Failure flow
- If the reminder level does not have automatic send enabled, nothing happens.
- If case creation fails, the standard error flow applies.

## Document upload flow

### Trigger
A case was just successfully created at Collectia.

### Main flow
1. Iterate reminder lines of type Customer Ledger Entry.
2. For each sales invoice: generate PDF via report, upload with metadata (FAK type).
3. For each service invoice: generate PDF via service invoice report, upload with metadata (FAK type).
4. For non-invoice lines: send metadata only with appropriate amount type.
5. After all lines, aggregate additional fee lines and send as single fee document.

### Failure flow
- SOAP errors during any document upload stop processing with error.

## Open Collectia case in browser

### Trigger
User clicks "Open Collectia case" on an issued reminder that has a case number.

### Main flow
1. Read setup to get the URL template.
2. Substitute customer number, username, password, and case number into the URL.
3. Open the URL in the default browser.

### Rebuild note
The URL currently contains credentials. This must be eliminated in the rebuild.
