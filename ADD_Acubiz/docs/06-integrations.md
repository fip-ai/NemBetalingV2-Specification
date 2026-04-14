# Integrations

## External contracts

### Azure Blob Storage — inbound file pickup
- Purpose: Retrieve CSV expense files, receipt images, and documents from Azure containers.
- Direction: inbound.
- Dependency abstraction: uses the standard BC Azure Blob Storage client components (ABS Blob Client, authorization, container content).

#### Operations
| Operation | Inputs | Expected outcome | Failure semantics |
|-----------|--------|------------------|-------------------|
| List blobs | storage account, container, shared key, optional subdirectory filter, optional name filter | enumeration of matching blobs with paging support | Azure error stops processing |
| Get blob as stream | blob full name | blob content as InStream | Azure error stops processing |
| Delete blob | blob full name | source blob removed | Azure error stops processing |
| Put blob (archive copy) | target path and stream | blob copied to archive location | Azure error stops processing |

#### Payload meaning
| Element / payload concept | Meaning | Required / optional |
|---------------------------|---------|---------------------|
| Acubiz CSV file | semicolon-delimited expense transaction rows | Required for expense import |
| Image/receipt file | JPEG/PDF attachment linked to an expense line by file name | Optional per journal line |
| XML invoice/credit note | UBL-formatted document with endpoint ID for GLN validation | Required when GLN check is enabled |

### Azure Blob Storage — outbound file delivery
- Purpose: Upload generated employee export workbooks to Azure.
- Direction: outbound.
- Dependency abstraction: same BC Azure Blob Storage client.

#### Operations
| Operation | Inputs | Expected outcome | Failure semantics |
|-----------|--------|------------------|-------------------|
| Put blob (export) | target path and Excel stream | workbook stored in Azure | Azure error stops processing |

### Email sending — Continia document forwarding
- Purpose: Send blob content as email attachment to Continia Document Capture email address.
- Direction: outbound.
- Dependency abstraction: BC Email framework with dedicated email scenario.

#### Operations
| Operation | Inputs | Expected outcome | Failure semantics |
|-----------|--------|------------------|-------------------|
| Send email with attachment | recipient from document category, subject from config, base64-encoded blob content, file name | email delivered | send failure stops processing with error |

#### Response meaning
| Response concept | Meaning | App reaction |
|------------------|---------|-------------|
| Email send success | file considered delivered | archive and delete source blob; pause briefly before next file |
| Email send failure | file not delivered | processing stops with error; source blob preserved |

### Continia Document Capture — document category lookup
- Purpose: Resolve the target email address for forwarded documents.
- Direction: inbound reference lookup.
- Dependency abstraction: reads CDC Document Category record.

### GLN validation from XML content
- Purpose: Verify that the XML document's customer endpoint identifier matches the company's GLN before forwarding.
- Direction: content inspection.

#### Operations
| Operation | Inputs | Expected outcome | Failure semantics |
|-----------|--------|------------------|-------------------|
| Extract endpoint ID from UBL Invoice or CreditNote | XML InStream | endpoint ID text extracted using namespace-aware XPath | no match or no extraction means file is skipped for forwarding |

## Events and extension points relevant to integrations
- No custom events are published by this app.
- The integration type enum is extensible, allowing future scenarios to be added by dependent apps.

## Integration posture
This app is a file-moving integration bridge. It does not implement a general-purpose API layer or publish events for external consumers. Its integration value is the reliable pickup-process-archive-delete cycle for Azure blobs.