# Solution Inventory

## Source Inventory Summary
| Object kind | Count | Notes |
|-------------|-------|-------|
| Tables | 2 | Setup table and Azure configuration table |
| Pages | 3 | Setup/configuration and approver list |
| Page extensions | 1 | General Journal action entry point |
| Codeunits | 2 | Azure import/email processing |
| XmlPorts | 2 | Expense import and employee export |
| Enums | 1 | Integration scenario catalog |
| Enum extensions | 1 | Email scenario extension |
| Permission sets | 1 | Production permission packaging |
| Diagnostics CSV files | several | static analysis artifacts, not runtime behavior |
| App packages in .alpackages | several | dependency binaries for compilation |

## Domain Breakdown
| Domain | Purpose | Included objects |
|--------|---------|------------------|
| Setup | Global app setup for journal destination | ADD Acubiz Setup table/page |
| AzureConfig | Per-scenario Azure Blob Storage and routing configuration | ADD Azure Config table/page, integration enum |
| ExpenseImport | Import Acubiz CSV expenses into general journals and attach supporting images | import codeunit, import xmlport, journal page extension |
| DocumentForwarding | Forward Azure files by email into Continia workflows | import/export codeunit, email scenario enum extension |
| EmployeeExport | Transform employee input into Acubiz export workbook and upload to Azure | employee export xmlport, approver page |
| Security | Access packaging | Acubiz Permission |

## Object Inventory
| Object kind | Object name | Domain | Purpose | Required for rebuild | Depends on |
|-------------|-------------|--------|---------|----------------------|------------|
| Table | ADD Acubiz Setup | Setup | Stores journal template and batch destination for imported transactions | Yes | General journal setup |
| Page | ADD Acubiz Setup | Setup | Maintains single-record app setup | Yes | setup table |
| Table | ADD Azure Config | AzureConfig | Stores Azure storage settings per integration scenario, including secret key in isolated storage | Yes | integration enum, Continia document category |
| Page | ADD Azure Configs | AzureConfig | Maintains per-scenario Azure configuration and secret key | Yes | Azure config table |
| Enum | ADD Integration Type | AzureConfig | Classifies supported integration scenarios | Yes | Azure config table and processing logic |
| Enum extension | ADD Email Scenario | DocumentForwarding | Adds dedicated email scenario for Continia-bound messages | Yes | BC email framework |
| Codeunit | ADD Import Acubiz Files | ExpenseImport | Pulls Acubiz CSV files from Azure and imports them into journals | Yes | setup table, Azure config, import xmlport |
| XmlPort | ADD Acubiz Import | ExpenseImport | Parses Acubiz CSV lines into general journal lines and imports receipt images | Yes | journals, Azure config, incoming documents, DSS shared tables |
| Page extension | ADD General Journal | ExpenseImport | Adds manual action to run Acubiz import from the General Journal page | Yes | import xmlport |
| Codeunit | ADD Imp. / Exp. Acubiz Files | DocumentForwarding | Reads configured blobs from Azure and optionally emails them to Continia, then archives/deletes them | Yes | Azure config, email framework, Continia categories |
| XmlPort | ADD Imp. Employee > Exp.Acubiz | EmployeeExport | Reads employee-style input, transforms it to Excel output, uploads the result to Azure | Yes | DSS setup, approver and department mapping tables, Excel buffer, Azure config |
| Page | ADD Acubiz Godkendere | EmployeeExport | Lists approvers and provides employee import/export actions | Yes | shared approver table, xmlports |
| Permission set | Acubiz Permission | Security | Grants access to app objects | Yes | all production objects |

## Non-code Assets
| Asset | Purpose | Rebuild relevance |
|-------|---------|-------------------|
| app.json | manifest and dependencies | High |
| diagnostics CSV files | code analysis reports | Low |
| dependency app packages | compile-time references | Medium |

## External/shared objects referenced but not owned here
- DSS Setup
- Acubiz Godkendere
- Acubiz Department Mapping
- CDC Document Category
- ABS Container Content and Azure Blob Storage client components
- Incoming Document and Incoming Document Attachment
These must be treated as external dependencies, not reconstructed inside this app unless a full-solution rebuild later requires them.