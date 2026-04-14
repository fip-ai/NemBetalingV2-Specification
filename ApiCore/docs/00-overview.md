# Overview

## App Identity
- App name: Api Core
- Publisher: Finn Pedersen France
- App role in solution: Foundation/shared infrastructure app
- Production app / test app / both covered: Both `Api Core` and `Api Core Test`

## Purpose
Api Core provides the common infrastructure for tracking, executing, scheduling, diagnosing, and testing outbound API operations inside the NemBetalingV2 solution. It is a technical foundation app rather than an end-user business domain app.

## Scope
Inside scope:
- persistent recording of API operations
- request/response payload storage
- operation status and result tracking
- pre-send validation rules
- success and failure classification
- scheduled background execution support
- debugging/manual execution support
- reusable JSON encode/decode helpers
- payload viewer support for request/response inspection
- a dedicated test app validating the operational contract

Outside scope:
- concrete business-domain API integrations
- business-specific endpoint catalogs beyond simple test fixtures
- setup/configuration for a specific external provider

## Personas
- Technical support user: reviews API history and diagnostics
- Developer or advanced consultant: debugs payloads and operation flow
- Dependent app developer: reuses the infrastructure for concrete integrations
- Test developer: validates expected behavior through the paired test app

## Dependencies
| Dependency | Type | Why it is needed |
|------------|------|------------------|
| Business Central base/system objects | platform | scheduled task records, page management, task scheduling, reflection, JSON/XML support |
| Api Core Test | paired test app relationship | validates internal contracts |
| Microsoft test libraries | test dependency | assertions and BC test framework support |

## Platform Posture
- Runtime: 16.0
- Application version: 27.0.0.0
- Platform: 1.0.0.0
- Target: production app currently omits explicit target, test app targets Cloud
- Resource exposure posture: permissive development posture, with debugging and source download enabled

## Rebuild Goal
A rebuilt version must preserve the foundation role of the app: durable API operation logging, clear send preconditions, deterministic request/response handling, background scheduling support, failure capture, payload inspection, and a test suite that defines the expected behavior. The rebuild may modernize structure and harden manifest posture, but must preserve the technical support and orchestration semantics.