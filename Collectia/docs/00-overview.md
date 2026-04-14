# Overview

## App Identity
- App name: Inkasso (business name: Collectia)
- Publisher: Nubiz A/S
- App role in solution: Standalone debt-collection integration app
- Production app / test app / both covered: Production app only (no test app exists)

## Purpose
This app automates the transfer of outstanding customer debt from Business Central's issued reminder process to Collectia's (formerly Kredinor's) external collection service via SOAP web services. When a reminder reaches a configured collection level, or when a user manually triggers it, the app creates a debt-collection case at Collectia and uploads supporting invoice documents.

## Scope
Inside scope:
- connection to Collectia SOAP web service (login, create case, send documents, activate case, logout)
- configuration of Collectia credentials, endpoints, and case-creation defaults
- extension of issued reminder headers with collection tracking fields
- extension of reminder levels with automatic-send configuration
- manual and automatic case-creation workflows
- invoice PDF generation and upload to Collectia
- non-PDF document metadata submission for other document types
- optional customer blocking upon case creation
- activity logging

Outside scope:
- the reminder process itself (owned by BC base application)
- payment reconciliation from Collectia
- Collectia case status monitoring or callback
- customer unblocking

## Personas
- Accounts receivable clerk: sends reminders to collection and reviews case status
- Finance administrator: configures Collectia connection and collection behavior
- Technical support: reviews activity logs and troubleshoots failures

## Dependencies
| Dependency | Type | Why it is needed |
|------------|------|------------------|
| Business Central base application | platform | customer records, issued reminders, reminder levels, sales/service invoices, report rendering, activity logging |
| Collectia SOAP web service (external) | external service | receives debt-collection cases and documents |

## Platform Posture
- Runtime: 3.0 (severely outdated)
- Application version: 14.0.0.0 (severely outdated)
- Platform: 14.0.0.0
- Target: Extension (should be Cloud in rebuild)
- Resource exposure posture: `showMyCode: false`

## Rebuild Goal
A rebuilt version must preserve the core workflow: creating Collectia collection cases from issued reminders, uploading supporting documents, and optionally blocking the customer. The rebuild must modernize architecture, security, testability, and compliance while keeping the same business outcome.