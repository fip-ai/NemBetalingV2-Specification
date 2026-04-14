# Integrations

## External contracts

### Collectia SOAP web service
- Purpose: Create and manage debt-collection cases at Collectia.
- Direction: outbound.
- Dependency abstraction: none in current source (hardcoded HTTP calls); **rebuild must use an interface**.

#### Operations
| Operation | SOAP action | Inputs | Expected outcome | Failure semantics |
|-----------|-------------|--------|------------------|-------------------|
| Login | WSLOGIN | customer number, username, password, language | session token returned | error text surfaced if return type is S or E |
| Create case | WSOPRSAG | session token, customer data, reminder data, summons text, status, course | case number returned | error text surfaced if return type is W; case number "0" is also failure |
| Upload document with file | WSOPRSD2 | session token, case number, document ID, base64 PDF, metadata | document registered | error text surfaced if return type is W or E |
| Upload document metadata only | WSOPRSD2 | session token, case number, document ID, amounts, dates | metadata registered | error text surfaced if return type is W or E |
| Activate case | WSAKTSAG | session token, case number | case activated | no explicit error handling evidenced |
| Logout | WSLOGOUT | session token, language | session ended | no explicit error handling evidenced |

#### Payload meaning — case creation
| Element | Meaning | Required / optional |
|---------|---------|---------------------|
| staevningsTekst | summons/claim description | Required |
| reference | BC customer number | Required |
| cvrNummer | Danish VAT registration number (digits only, max 8) | Required for domestic companies |
| vatNo | foreign VAT number | Required for foreign entities |
| klientNummer | Collectia client/customer number from setup | Required |
| type | PER (person), VRK (company), or UDL (foreign) | Required |
| navn, coNavn, adresse, postNummer, land | debtor address details | Required |
| valuta | currency code (DKK for domestic) | Required |
| status | INK or RYK — collection mode | Required |
| forloeb | course/workflow code from setup | Required |
| baggrund | background type — always FAK (invoice-based) | Fixed |

#### Payload meaning — document upload
| Element | Meaning | Required / optional |
|---------|---------|---------------------|
| sagsNummer | Collectia case number | Required |
| id | BC document number | Required |
| filNavn | PDF file name | Required for PDF uploads, empty for metadata-only |
| filData | base64-encoded PDF content | Required for PDF uploads, empty for metadata-only |
| filDataFormat | B64 for PDF, TXT for metadata-only | Required |
| beloebType | FAK/BET/KRE/RNT/GEB — amount classification | Required |
| beloeb | amount value | Required |
| dato | posting date (YYYYMMDD) | Required |
| forfald | due date (YYYYMMDD) | Required |

#### Response meaning
| Response element | Meaning | App reaction |
|------------------|---------|-------------|
| returType = S or E | login/operation error | raise error with returTekst |
| returType = W | warning/error on case or document operation | raise error with returTekst |
| sagsNummer = 0 | case creation failed | raise error with returTekst |
| sagsNummer > 0 | success | store case number, proceed to document upload |
| billet | session token | stored for subsequent requests |

## Events

### Subscribed events
| Event | Source | Purpose |
|-------|--------|---------|
| OnAfterIssueReminder | Codeunit Reminder-Issue | triggers automatic case creation when reminder level is configured for it |

### Published events
None. **Rebuild should publish events at key decision points**.

## Integration posture
The app is a one-way outbound integration. It pushes data to Collectia but does not receive callbacks or status updates. All communication is synchronous SOAP over HTTP.
