# Business Central App Quality Review: Collectia (Inkasso)

## Executive Summary
- **Context / scope**: Full review of the Collectia debt-collection integration app (`Inkasso`) published by Nubiz A/S. 10 AL files, no test app, no CI pipeline, targeting BC platform 14.0 / runtime 3.0.
- **Overall health rating**: 🔴 **Red** — the app works functionally but has critical architectural, security, quality, and maintainability issues that make it unsuitable for modern deployment without a rewrite.
- **Top 3 strengths**:
  1. Clear business purpose: automates debt-collection case creation via SOAP web service.
  2. Activity logging infrastructure exists for troubleshooting.
  3. Bilingual captions (Danish/English) present on most objects.
- **Top 3 risks / blockers**:
  1. Credentials stored in plain text table fields.
  2. No test app, no analyzers, no CI pipeline.
  3. Severely outdated architecture — legacy C/AL patterns, massive monolithic codeunit, no separation of concerns.

## Solution Overview
- **App name**: Inkasso
- **Publisher**: Nubiz A/S
- **Version**: 18.3.220930.0
- **Target**: Extension (no explicit Cloud/OnPrem declaration)
- **Deployment type**: Likely PTE (no AppSource metadata)
- **Dependencies**: None declared (standalone)
- **Runtime**: 3.0 (very old — current is 16.0+)
- **Application**: 14.0.0.0 (very old — current is 26.0+)
- **Object footprint**:
  - Tables: 1
  - Table extensions: 2
  - Pages: 2
  - Page extensions: 2
  - Codeunits: 2
  - Enums: 1
  - Permission sets: 0
  - Test objects: 0
  - Reports: 0
  - XMLports: 0
  - Interfaces: 0

## Findings by Area

### Architecture & Design

**Severity: Blocker**

- **Monolithic web-service codeunit**: The main codeunit (`IIT_Kredinor WS`) is ~500 lines containing authentication, case creation, document sending (3 near-identical variants), HTTP transport, SOAP envelope construction, XML response parsing, PDF generation, invoice-copy logic, activity logging, and customer blocking — all in one object with no separation of concerns.
- **No interfaces or abstractions**: HTTP transport is hardcoded inline. No way to stub or test any of it.
- **Copy-paste duplication**: `SendDocumentFAK` and `SendDocumentServiceFAK` are nearly identical procedures differing only in the source record type. `SendDocumentOther` is a third variant. The same SOAP response-error checking block is copy-pasted 6+ times.
- **SOAP envelope construction by string concatenation**: All requests are built by concatenating XML strings. No template, no helper, no structured approach. Extremely fragile and hard to maintain.
- **Legacy naming**: Object names use `IIT_` prefix and internal codename "Kredinor" while the business name is "Collectia". Captions are updated but code identifiers are not.
- **No domain-first folder structure**: Objects are organized by type (`codeunit/`, `page/`, etc.) rather than by domain.
- **Global state**: The `Token` variable and `KredinorSetup` record are instance-level state on the WS codeunit, creating hidden coupling between procedures.
- **Legacy `Option` field in page**: The input dialog page uses a local `Option` variable (`Status: Option Inkasso,Rykkerservice`) instead of referencing the existing enum.
- **`showMyCode: false`**: Source is hidden from debugging, which is hostile to customers and partners.

### Code Quality & Standards

**Severity: Blocker**

- **No `NoImplicitWith` feature**: The manifest only declares `TranslationFile`. This means the app uses implicit `with` binding, which is removed in modern AL.
- **Legacy C/AL syntax throughout**: `FINDSET`, `FIND('-')`, `NEXT = 0`, `CALCFIELDS(Rec.Field)`, `MODIFY` without `true`, uppercase keywords — all indicate a mechanical conversion from C/AL rather than a proper AL rewrite.
- **Hardcoded SOAP XML**: fragile and unmaintainable.
- **Missing `DataClassification` on most fields**: The setup table fields lack `DataClassification`. This is a compliance violation for GDPR.
- **No captions on enum values**: The `Begin Case in Default ENUM` has values `INK` and `RYK` with no captions at all.
- **Commented-out code**: Multiple commented-out lines in the WS codeunit, including old DotNet references and alternative implementations.
- **Save-log comments in source**: The enum file contains `#SaveLog` metadata from an old version-control system — dead weight.
- **ID range is absurdly wide**: `50000–99000` (49,001 IDs) for an app that uses fewer than 15 objects. This is wasteful and potentially conflicting.
- **Inconsistent object numbering**: Table is 60000, pages are 60000–60001, codeunits are 60000–60001, enum is 60000, but table extensions are 60296–60297 and page extensions are 61432–61438. No coherent allocation strategy.

### Data & Upgrade

**Severity: Major**

- **No upgrade codeunit**: If the schema changes, there is no data migration strategy.
- **No install codeunit**: Setup record creation is page-triggered only.
- **Table extension fields use IDs in 50000 range**: This is outside the declared app ID range of 50000–99000, which is technically valid but poorly organized.
- **`Description = 'INV1.00'`**: Used as a version marker on extension fields — non-standard and confusing.

### Performance & Telemetry

**Severity: Major**

- **No telemetry**: No `Session.LogMessage` calls. The activity log is better than nothing but is not telemetry.
- **Synchronous HTTP calls in user context**: All SOAP calls happen synchronously in the foreground. No background processing, no timeout handling beyond a hardcoded 20-second client timeout.
- **`Commit()` inside logging**: The `CreateLogFile` procedure calls `Commit()`, which is dangerous inside error-handling flows and transaction boundaries.
- **No `SetLoadFields`**: Record loading is unoptimized.

### Security & Permissions

**Severity: Blocker**

- **Credentials stored as plain text**: `WS Username` and `WS Password` are stored in regular `Text[30]` fields in the setup table. They are displayed unmasked on the setup page. This is a critical security violation.
- **No permission set**: The app declares zero permission sets. Users must be granted SUPER or manually configured.
- **Credentials passed in URL via HYPERLINK**: The "Open Collectia case" action constructs a URL containing username and password as parameters, then opens it in the browser. This exposes credentials in browser history, logs, and potentially network traffic.
- **No isolated storage**: Azure Key Vault or isolated storage should be used for secrets.

### Testing & Automation

**Severity: Blocker**

- **No test app**: Zero tests.
- **No CI/CD pipeline**: No `.github/`, `.AL-Go/`, or pipeline configuration.
- **No analyzers configured**: No `.vscode/settings.json` with analyzer settings.
- **Untestable architecture**: The monolithic codeunit with hardcoded HTTP calls cannot be unit-tested without interfaces and stubs.

### Compliance (AppSource / PTE)

**Severity: Major**

- **Not AppSource-ready**: Missing `privacyStatement`, `EULA`, `contextSensitiveHelpUrl`, permission sets, entitlements, telemetry, and analyzers.
- **`target: Extension`**: Should be `Cloud` for modern deployments.
- **Missing `features`**: `NoImplicitWith` and `GenerateCaptions` are not declared.
- **Extremely old runtime/application target**: 3.0 / 14.0 is from 2019.

### Documentation & Supportability

**Severity: Major**

- **No README**: No documentation in the repository.
- **No AGENTS.md or workspace instructions**.
- **Activity log is the only diagnostics aid**: Better than nothing, but insufficient.
- **Version displayed on setup page is hardcoded as a label**: Not derived from `app.json`.

## Analyzer & Test Results
- **CodeCop**: Not configured. Expected many violations.
- **AppSourceCop**: Not configured. Not applicable in current state.
- **Custom analyzers**: None.
- **Tests**: None.

## Recommendations

| # | Area | Severity | Effort | Recommendation |
|---|------|----------|--------|----------------|
| 1 | Security | Blocker | Medium | Move credentials to isolated storage immediately. Never store passwords in plain table fields. |
| 2 | Security | Blocker | Low | Remove credentials from HYPERLINK URL construction. |
| 3 | Architecture | Blocker | High | Break monolithic WS codeunit into: setup management, HTTP transport, SOAP message builder, response parser, case orchestrator, document sender. |
| 4 | Architecture | Blocker | Medium | Introduce interface for HTTP transport so the app becomes testable. |
| 5 | Testing | Blocker | High | Create a paired test app with acceptance tests for case creation, document sending, validation rules, and error handling. |
| 6 | Code quality | Blocker | Medium | Add `NoImplicitWith` feature, fix all implicit `with` usage, add `DataClassification` to all fields, add captions to enum values. |
| 7 | Security | Major | Low | Create a proper permission set. |
| 8 | Architecture | Major | Medium | Eliminate copy-pasted SOAP response checking and document-sending variants. |
| 9 | Data | Major | Low | Add install and upgrade codeunits. |
| 10 | Performance | Major | Medium | Move SOAP calls to background task execution. |
| 11 | Compliance | Major | Low | Update runtime, application, target, and manifest metadata to modern standards. |
| 12 | Code quality | Minor | Low | Remove commented-out code, save-log artifacts, and legacy DotNet references. |
| 13 | Code quality | Minor | Low | Rationalize object ID allocation. |
| 14 | Documentation | Minor | Low | Add README with setup and usage instructions. |

## Next Steps

### Quick wins (< 2 weeks)
- Move credentials to isolated storage.
- Remove credentials from URL.
- Add permission set.
- Add `NoImplicitWith`, `DataClassification`, enum captions.
- Add install/upgrade codeunits.
- Clean up commented-out code.

### Strategic items (> 2 weeks)
- Full architectural restructuring with interfaces and separation of concerns.
- Build paired test app.
- Add CI/CD pipeline with analyzers.
- Move to background processing for SOAP calls.
- Consider a full clean-room rebuild using `bc-reverse-engineer` + `bc-app-builder`.

### Recommendation
This app is an ideal candidate for a **full clean-room rebuild**. The functional scope is small and well-defined, but the implementation quality is so poor that incremental fixes would be more expensive than starting fresh from a proper specification.
