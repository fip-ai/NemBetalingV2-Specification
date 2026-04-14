# Overview

## App Identity
- App name: ADD_Acubiz
- Publisher: ADDvision ApS
- App role in solution: Integration app for Acubiz expense files, Azure Blob Storage transport, journal import, attachment import, and employee export
- Production app / test app / both covered: Production app only

## Purpose
This app connects Business Central to Acubiz-related file exchanges. It imports expense transactions from Azure Blob Storage into general journal lines, imports associated receipt images as incoming document attachments, exports employee data for Acubiz consumption, and routes selected files by email to Continia Document Capture.

## Scope
Inside scope:
- Acubiz setup for journal destination
- Azure storage configuration per integration scenario
- CSV-based expense import
- picture attachment import from Azure Blob Storage
- employee-list transformation and export to Azure Blob Storage
- blob-to-email forwarding for Continia scenarios
- UI entry points for setup and manual processing

Outside scope:
- full employee master maintenance
- full document-capture processing logic inside Continia
- generic Azure storage framework implementation
- general DSS configuration and mapping tables owned by other apps

## Personas
- Finance administrator: configures journal destination and runs imports
- Integration administrator: configures Azure storage endpoints and credentials
- Accounts payable or document-processing staff: forwards or imports external documents
- Technical support user: troubleshoots integration setup and file movement

## Dependencies
| Dependency | Type | Why it is needed |
|------------|------|------------------|
| ADD_General | custom dependency | provides DSS setup, Acubiz employee/approver-related tables, department mapping, and likely shared infrastructure |
| Continia Document Capture | external app dependency | provides document categories and downstream email/document handling |
| Business Central base/system/app libraries | platform | journals, vendors, G/L accounts, company information, XML, email, incoming documents, Excel buffer, Azure Blob Storage client |

## Platform Posture
- Runtime: 15.0
- Application version: 26.0.0.0
- Platform: 1.0.0.0
- Target: not explicitly stated in manifest
- Resource exposure posture: permissive development posture with debugging and source download enabled

## Rebuild Goal
A rebuilt version must preserve the operational contract: configurable Azure-based file exchange, reliable journal import from Acubiz CSV files, attachment import for journal lines, employee export generation, and selective blob-to-email forwarding for Continia. The rebuild may modernize architecture and harden configuration/security, but must preserve the file-processing semantics and business outcomes.