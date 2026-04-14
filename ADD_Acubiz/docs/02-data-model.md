# Data Model

## Tables

### ADD Acubiz Setup
- Purpose: Stores the destination journal location used when Acubiz expense transactions are imported.
- Persistence role: Single-record application setup.
- Record lifecycle summary: The setup page ensures a record exists; users maintain journal template and batch values there.

#### Fields
| Field | Type | Required | Default | Editable | Business meaning | Validation / constraints |
|------|------|----------|---------|----------|------------------|--------------------------|
| Primary Key | Code up to 10 | Yes for persisted record | blank if auto-created without explicit key assignment | effectively system-managed | singleton-record anchor | clustered primary key; current source relies on implicit singleton behavior |
| Journal Template Name | Code up to 10 | Yes for functional import | blank | Yes | general journal template to receive imported expense lines | when changed, journal batch name is cleared |
| Journal Batch Name | Code up to 10 | Yes for functional import | blank | Yes | general journal batch to receive imported expense lines | must belong to selected journal template |

#### Relationships
| Field / concept | Related entity | Relationship type | Meaning |
|-----------------|----------------|-------------------|---------|
| Journal Template Name | General Journal Template | lookup | determines journal family for imports |
| Journal Batch Name | General Journal Batch | dependent lookup | determines exact batch inside selected template |

#### Keys and uniqueness
- Primary key: Primary Key.
- Additional keys: none.
- Uniqueness semantics: intended as a singleton setup record.

#### Lifecycle rules
- On create/open: if the setup record does not exist, the page creates it automatically.
- On update: changing journal template clears the existing batch selection.
- On delete: no explicit source restriction evidenced, but page disallows delete from UI.

### ADD Azure Config
- Purpose: Stores Azure Blob Storage configuration and routing behavior per integration scenario.
- Persistence role: One configuration row per integration type.
- Record lifecycle summary: Maintained manually through a list page; secret key is stored separately in company isolated storage.

#### Fields
| Field | Type | Required | Default | Editable | Business meaning | Validation / constraints |
|------|------|----------|---------|----------|------------------|--------------------------|
| Integration Type | Enum | Yes | none | Yes | identifies which integration scenario this row configures | primary key |
| Storage Account Name | Text up to 100 | Yes for active integration | blank | Yes | Azure storage account name | none evidenced beyond runtime need |
| Container Name | Text up to 100 | Yes for active integration | blank | Yes | Azure blob container name | none evidenced beyond runtime need |
| Storage Subdirectory | Text up to 500 | Often required | blank | Yes | folder/prefix containing source or target files | used for filtering/selecting blobs |
| Storage Archive Subdirectory | Text up to 500 | Optional | blank | Yes | folder/prefix for processed-file archive copy | if blank, files may simply be deleted after processing |
| Subject | Text up to 250 | Optional/required for email-routing scenarios | blank | Yes | email subject used when forwarding blobs by email | only used for email-forwarding integrations |
| Document Category | Code up to 10 | Optional/required for email-routing scenarios | blank | Yes | Continia document category controlling recipient email | when present, forwarding-by-email path is used |
| Full Name Filter | Text up to 100 | Optional | blank | Yes | blob full-name filter for selective processing | narrows blob processing set |
| Check GLN | Boolean | Optional | false | Yes | enables recipient GLN validation before email forwarding | only relevant for XML documents sent by email |

#### Relationships
| Field / concept | Related entity | Relationship type | Meaning |
|-----------------|----------------|-------------------|---------|
| Integration Type | ADD Integration Type | classification | determines processing behavior |
| Document Category | CDC Document Category | lookup | determines recipient email address for Continia forwarding |
| Isolated password | company isolated storage | secret association | stores Azure shared key outside table fields |

#### Keys and uniqueness
- Primary key: Integration Type.
- Additional keys: none.
- Uniqueness semantics: at most one configuration per integration scenario.

#### Lifecycle rules
- Secret key is not stored in a normal field. It is replaced and retrieved via isolated company-scoped storage.
- Setting a new secret deletes the previous secret for that scenario before storing the new one.

## Enums and options

### ADD Integration Type
- Purpose: Catalog of supported integration scenarios.
- Extensible: Yes.
- Used by: Azure configuration and processing code.

| Value | Meaning | Business effect |
|-------|---------|-----------------|
| Acubiz CSV Import | inbound transaction import | source for expense CSV pickup |
| Acubiz Pictures Import | inbound image import | source for receipt/attachment pickup |
| Acubiz Employee Export | outbound employee-file delivery | target for generated export workbook |
| Import / Email to Continia | inbound blob forwarding by email | files are emailed to Continia destination and optionally archived |
| Import / Email to Continia 2 | second independent forwarding channel | same logic with separate configuration |
| Import / Email to Continia 3 | third independent forwarding channel | same logic with separate configuration |

### Email scenario extension: ADD Continia Emails
- Purpose: Dedicated email-sending context for this integration.
- Extends: the base email scenario catalog.
- Rebuild note: preserve a dedicated scenario so email routing and policies can be isolated from generic emails.

## Extensions

This app does not define table extensions in its own data model. It uses a page extension only.