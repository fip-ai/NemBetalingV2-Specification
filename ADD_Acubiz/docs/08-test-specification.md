# Test Specification

## Test app role
No paired test app exists in the source repository. A rebuilt version must include a paired test app to validate the integration contract.

## Required test structure
- Required domains mirrored in test app:
  - Setup
  - AzureConfig
  - ExpenseImport
  - DocumentForwarding (where stubbable)
- Required helpers/stubs/spies:
  - stub for Azure Blob Storage client (to avoid real Azure calls)
  - stub for Email framework (to avoid real email sends)
  - test helper for creating setup and Azure configuration fixtures
- Required permission artifacts:
  - test permission set

## Acceptance criteria
| ID | Scenario | Given | When | Then |
|----|----------|-------|------|------|
| AC-SETUP-001 | Setup auto-creates on open | no setup record | page opens | a record is created |
| AC-SETUP-002 | Changing template clears batch | setup with template A and batch B | template changed to C | batch is blank |
| AC-AZ-001 | Secret round-trips via isolated storage | an Azure config row | secret is set and retrieved | retrieved secret matches stored value |
| AC-AZ-002 | Missing config stops processing | no Azure config for CSV import | import codeunit runs | error is raised |
| AC-IMP-001 | CSV row creates journal line for finance account | valid CSV row with finance account marker | import xmlport processes it | journal line exists with correct G/L account, amount, dates, and document number |
| AC-IMP-002 | CSV row creates journal line for vendor account | valid CSV row with vendor account marker | import xmlport processes it | journal line exists with correct vendor account and amount |
| AC-IMP-003 | Wrong company aborts import | CSV row with different company name | import xmlport processes it | import is marked unsuccessful |
| AC-IMP-004 | Dimensions are assigned when present | CSV row with operating-area and payroll-area values | import xmlport processes it | journal line has correct dimension set |
| AC-IMP-005 | Missing dimension values do not crash import | CSV row with blank dimension columns | import xmlport processes it | journal line is created without extra dimensions |
| AC-FWD-001 | GLN match allows forwarding | XML invoice with matching GLN | GLN check runs | file is accepted for sending |
| AC-FWD-002 | GLN mismatch skips forwarding | XML invoice with non-matching GLN | GLN check runs | file is not sent |
| AC-FWD-003 | Missing document category skips email | Azure config without document category | forwarding processes file | file is archived/deleted without email attempt |
| AC-EXP-001 | Invalid header stops export | input file with wrong header | export xmlport runs | error is raised |
| AC-EXP-002 | Department mapping enriches output | valid input row with mapped department | export processes it | output row contains mapped department value |
| AC-EXP-003 | Approver lookup enriches output | valid input row with known approver | export processes it | output row contains approver payroll number |

## Mandatory negative tests
- missing Azure configuration
- wrong company in CSV row
- invalid header format in employee file
- GLN mismatch
- email send failure (requires email stub)

## Minimum regression suite
- setup singleton behavior
- journal template / batch dependency
- Azure secret management
- CSV-to-journal-line mapping for both account types
- company filter enforcement
- dimension assignment
- GLN extraction and validation
- employee export header validation
- department and approver enrichment

## Rebuild note
The original app has no tests. Producing a paired test app is a critical improvement over the legacy implementation. Azure and email dependencies must be abstracted behind interfaces so tests can run without real external systems.