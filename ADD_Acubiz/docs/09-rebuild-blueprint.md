# Rebuild Blueprint

## App manifest blueprint
- App name: ADD_Acubiz
- Test app name: ADD_Acubiz Test (new in rebuild)
- App role: Azure/file-integration bridge for Acubiz expense processing and related document flows
- Dependencies:
  - shared ADD_General app for DSS and Acubiz reference tables
  - Continia Document Capture for document categories and downstream document processing
  - BC email, incoming document, Azure Blob Storage, Excel, and journal framework
- Runtime/target posture:
  - rebuild for modern BC Cloud posture
  - preserve integration scenarios and external table dependencies

## Reconstruction order
1. Build setup table/page and Azure config table/page.
2. Build integration type enum and dedicated email scenario extension.
3. Build interfaces/wrappers for Azure Blob Storage and email so processing becomes testable.
4. Build CSV import pipeline and journal-line transformation logic.
5. Build attachment-import logic for receipt images.
6. Build blob-forwarding logic for Continia email scenarios.
7. Build employee transformation/export pipeline.
8. Build page extension and permission set.
9. Build paired test app with Azure/email stubs and acceptance tests.

## Object blueprint by domain

### Setup
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Table | Acubiz setup | singleton setup storing journal template and journal batch |
| Page | Acubiz setup page | admin card page that auto-creates setup record |

### Azure configuration
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Enum | Integration type | scenario catalog for CSV import, picture import, employee export, and three forwarding channels |
| Table | Azure config | per-scenario storage settings with isolated secret handling |
| Page | Azure config list | maintenance UI with masked secret update |
| Enum extension | dedicated email scenario | isolates email sends for this integration |

### Expense import
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Service/codeunit | CSV import orchestrator | picks up CSV blobs, imports them, archives/deletes on success |
| Parser/import component | CSV-to-journal transformer | maps rows into general journal lines and dimensions |
| Service/component | attachment importer | links receipt/image blobs as incoming documents |
| Page extension | General Journal action | manual launch point from journal UI |

### Document forwarding
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Service/codeunit | forwarding orchestrator | enumerates forwarding configs, optionally validates GLN, sends email, archives/deletes blobs |

### Employee export
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Page | Approver list page | display shared approvers and run employee actions |
| Import/export component | employee transformation/export | validates input header, enriches rows, writes Excel workbook, uploads to Azure |

### Security
| Object kind | Object name | Must implement |
|-------------|-------------|----------------|
| Permission set | Acubiz permission | grants access to all production objects, including forwarding orchestrator |
| Permission set | Test permission | grants test execution artifacts |

## Mandatory behaviors
- Setup is singleton-like and auto-created from UI.
- Azure secrets are stored outside normal table fields.
- CSV expense files create general journal lines in the configured template/batch.
- Company mismatch prevents importing the file.
- Receipt/image blobs are linked to created journal lines as incoming documents when available.
- Successfully imported or forwarded blobs are archived when archive path exists and deleted from source.
- GLN validation gates email forwarding when enabled.
- Employee export validates input header before processing data.
- Employee export generates a fixed-column Excel workbook and uploads it to Azure.

## Mandatory tests for rebuilt solution
- Implement all acceptance criteria from `08-test-specification.md`.
- Add interface-based stubs for Azure and email.
- Add tests for archive/delete behavior decisions.
- Add tests for image file-name normalization when wildcard markers appear.

## Non-negotiables
- Do not store Azure shared keys in plain table fields.
- Do not remove company-filter protection from CSV import.
- Do not collapse all integration scenarios into one undifferentiated config row.
- Do not perform real Azure or email I/O in tests.
- Do not omit a paired test app in the rebuild.

## Allowed modernization
- Replace xmlport-centric orchestration with cleaner parser/service architecture if behavior is preserved.
- Introduce telemetry and processing logs.
- Replace immediate `Error`-driven flow with richer result objects where user-visible behavior stays clear.
- Replace implicit singleton setup mechanics with a clearer setup pattern.
- Harden permissions, manifest metadata, and Cloud target posture.