# Domain Behavior

## Setup management

### Invariants
- The app expects a single setup record for journal import destination.
- Expense import cannot function meaningfully without journal template and journal batch setup.

### Core rules
- Opening the setup page auto-creates the setup record when missing.
- Changing the selected journal template invalidates the previous journal batch selection.

## Azure configuration and secret management

### Invariants
- Each integration scenario has at most one Azure configuration row.
- Azure shared keys are stored outside the table in isolated storage.

### Core rules
- Updating the visible masked password field replaces the isolated secret for the current integration row.
- When a stored secret exists, the page displays a masked placeholder rather than the true key.
- Processing code fails fast when required Azure configuration is missing.

## Acubiz CSV expense import

### Invariants
- Only CSV files from the configured Azure location are eligible.
- Journal lines are created in the configured template and batch.
- Import success is file-level, and successful files are archived and deleted from source.

### Validations and rejections
| Scenario | Rejection / prevention | Business reason |
|----------|------------------------|-----------------|
| Missing Azure config for CSV import | processing errors immediately | import cannot access storage |
| CSV row belongs to a different company | import aborts and file is marked unsuccessful | prevents posting another company’s expenses |
| Unsupported or missing setup values | downstream journal import will fail | journal target must exist and be valid |

### Core rules
- Import starts from the configured journal template and batch.
- If a baseline journal line does not exist at the expected starting line, a new line context is prepared.
- Each CSV row creates a new journal line with posting date, document date, document number, amount, description/comment, account information, and dimensions.
- Account interpretation depends on account-type marker in the file.
- A special fallback comment flow exists: when one finance text buffer is populated, it is reused on the next line instead of the combined text of the current row.
- After each journal line is created, the app attempts to import a matching image/attachment from the picture-import Azure location.

### Result semantics
- Import success means the xmlport completed without disqualifying the file and the processing code then archives and deletes the source blob.
- Import failure means the source blob is left in place.

## Receipt/image attachment import

### Invariants
- Imported expense lines may be linked to incoming documents.
- Image lookup is based on file name from the CSV row, with trailing wildcard suffix handling.

### Core rules
- For a matched image blob, the app creates an incoming document marked as released and of journal type.
- The created incoming document is linked to the journal line.
- A primary attachment is created from the blob stream.
- The processed image is archived and then removed from the source location.

## Blob forwarding to Continia by email

### Invariants
- Three separate integration types can run the same forwarding logic.
- If no matching forwarding configuration exists, processing is blocked.
- A blob may be forwarded directly by email or simply archived/deleted depending on configuration.

### Validations and rejections
| Scenario | Rejection / prevention | Business reason |
|----------|------------------------|-----------------|
| Missing forwarding Azure config | processing errors immediately | no source configuration exists |
| Email send fails | processing errors | file must not be treated as delivered |
| GLN check enabled and XML endpoint does not match company GLN | file is not sent | prevents routing incorrect documents into the company workflow |

### Core rules
- The forwarding process iterates configured forwarding scenarios 1 through 3.
- Each scenario enumerates blobs under the configured subdirectory, optionally filtered by full name.
- If a document category is configured, files are sent by email to the email address defined by that category.
- If GLN checking is enabled, XML content is inspected and only sent if the customer endpoint identifier matches the company GLN.
- Successfully handled files may be copied to an archive directory and are then deleted from the source location.
- If no document category is configured, files are archived/deleted without email sending.
- A short delay is inserted after successful email send.

## Employee transformation and export

### Invariants
- Input file must have a specific header structure.
- Export content is company-specific.
- Department and approver data are enriched from shared reference tables.

### Validations and rejections
| Scenario | Rejection / prevention | Business reason |
|----------|------------------------|-----------------|
| Header row does not match expected format | processing errors | input file is not in the supported Acubiz employee format |
| Company code in DSS setup is unsupported | processing errors | export rules are company-specific |
| Missing Azure config for employee export | processing errors | no export destination is configured |

### Core rules
- The first row is treated as a header validation row and skipped after validation.
- Incoming text fields are cleaned of carriage-return/line-feed artifacts.
- Certain company-name variants are normalized.
- Department values are mapped through a department-mapping table, with a couple of hard-coded overrides for specific management employees.
- Approver references are resolved through a shared approver table.
- Each source row becomes one row in an Excel workbook with a fixed column layout expected by Acubiz.
- When import finishes, the workbook is written to Azure Blob Storage under the configured export path.
