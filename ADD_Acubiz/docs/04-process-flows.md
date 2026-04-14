# Process Flows

## Maintain setup

### Trigger
A finance or integration administrator opens the setup page.

### Main flow
1. Ensure the singleton setup record exists.
2. Show journal template and journal batch fields.
3. If the journal template changes, clear the batch so the user must reselect a matching batch.

## Import Acubiz transactions from Azure

### Trigger
A user runs the import codeunit or uses the General Journal page action.

### Main flow
1. Read Acubiz setup to determine target journal template and batch.
2. Read Azure configuration for the Acubiz CSV import scenario.
3. Connect to Azure Blob Storage using the stored account name, container, and isolated secret key.
4. Enumerate blobs in the configured source subdirectory, limited to CSV files.
5. For each CSV file, download the blob stream.
6. Run the Acubiz import xmlport against the stream using the configured journal destination.
7. If import succeeds, optionally copy the source blob to the archive subdirectory.
8. Delete the processed source blob.

### Alternate flows
- If archive subdirectory is blank, processed files are simply deleted.

### Failure flow
1. If Azure configuration is missing or Azure operations fail, stop with an error.
2. If the xmlport determines the file is invalid for the current company or otherwise unsuccessful, do not archive/delete the source blob.

## Transform one CSV row into a journal line

### Trigger
The import xmlport processes one source record.

### Main flow
1. Determine whether the row belongs to the current company based on DSS setup and the company column in the file.
2. Create a new general journal line in the configured template and batch.
3. Assign posting date and document date from the file.
4. Use the Acubiz unique identifier as document number.
5. Interpret account type marker:
   - finance account marker leads to a G/L account posting
   - creditor marker leads to a vendor posting
6. Set VAT posting group when relevant.
7. Set amount from the domestic-currency amount field.
8. Build comment and description text from combined text and unique identifier, with a finance-text carry-over behavior when applicable.
9. Insert the journal line.
10. Add optional dimensions for operating area and payroll area when values are present.
11. Import the related image/receipt attachment if available.

### Failure flow
- If the file’s company does not match the current company, the xmlport marks the import unsuccessful and stops processing the file.
- Missing or invalid supporting master data may cause downstream validation errors in standard BC operations.

## Import related image/receipt attachment

### Trigger
A journal line has just been created from an imported CSV row.

### Main flow
1. Read Azure configuration for the Acubiz picture-import scenario.
2. Normalize the expected file name if wildcard syntax is present.
3. Search the configured blob path for the image file.
4. If found, download the image stream.
5. Create an incoming document for the journal line.
6. Create an incoming document attachment from the image stream.
7. Link the incoming document back to the journal line.
8. Archive the image blob if an archive path exists.
9. Delete the source image blob.

### Alternate flows
- If no image file is found, the journal line remains without an incoming document attachment.

## Forward Azure documents to Continia by email

### Trigger
A user or job queue runs the import/export codeunit.

### Main flow
1. Load company information and setup.
2. Load all Azure configuration rows for the three Continia forwarding scenarios.
3. For each configured scenario, connect to Azure Blob Storage.
4. Enumerate blobs under the configured source subdirectory, optionally filtered by full name.
5. For each blob, download the stream.
6. If a document category is configured:
   - optionally validate recipient GLN from XML content against company GLN
   - build an email addressed to the document category’s configured email address
   - attach the blob content as base64
   - send the email using the dedicated Acubiz/Continia email scenario
7. If processing was accepted, archive the blob if configured and delete the source blob.
8. If no document category is configured, archive/delete without email sending.

### Failure flow
- Missing configuration stops processing.
- Failed Azure operations stop processing.
- Failed email sending stops processing and prevents deletion.
- Failed GLN validation causes the file to be skipped for sending.

## Export employee data for Acubiz

### Trigger
A user runs the employee export xmlport from the approver page.

### Main flow
1. Determine the current company’s supported export name from DSS setup.
2. Validate that the input file header matches the expected format.
3. Skip the header row after validation.
4. Clean and normalize source fields.
5. Normalize company name variants.
6. Map department values using the shared mapping table and a few hard-coded overrides.
7. Resolve approver payroll number from the shared approver table.
8. Write one transformed row into an Excel buffer with the fixed Acubiz column layout.
9. After all rows are processed, create an Excel workbook.
10. Upload the workbook to Azure Blob Storage using the employee-export configuration.

### Failure flow
- Unsupported company or wrong input format causes immediate error.
- Missing Azure export configuration causes immediate error.
