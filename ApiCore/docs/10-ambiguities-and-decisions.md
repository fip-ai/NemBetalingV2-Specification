# Ambiguities and Decisions

## Certain
- ApiCore is intended as a shared technical foundation app.
- Api Operation records are non-deletable and sequentially numbered.
- Send preconditions depend on endpoint presence and method/payload compatibility.
- Any 2xx response is treated as success.
- Connection failure and non-2xx response are different result classes.
- Bulk scheduling targets operations with status New.
- The paired test app is an essential part of the intended contract.

## High-confidence inference
- The blank status/result values act as neutral initialization states rather than meaningful business states.
- The Line Style field is intended for UI presentation support but is not functionally important in the current source.
- The shared JSON result enum is either future-facing or lightly used by surrounding apps, even though it has no strong current runtime evidence in ApiCore itself.
- The viewer assets are included to improve human troubleshooting rather than to serve end-user workflows.

## Ambiguous
- The exact intended semantics of the In Progress, On Hold, and Cancelled statuses are not fully exercised by the current production code or test suite.
- The exact partial-failure behavior of bulk scheduling is not fully specified when some tasks schedule successfully before one fails.
- The production app’s missing explicit target in `app.json` may be intentional legacy carry-over or an omission.
- The full set of dependent apps expected to implement the request interface is outside the scope of ApiCore itself.
- Whether the Json result enum should remain as-is in a rebuilt version depends on surrounding app usage.

## Required builder decisions
| Topic | Safe decision | Rationale |
|-------|---------------|-----------|
| Production app target | Set explicit Cloud target | Matches test app posture and modern BC expectations |
| Status values not actively used | Preserve enum values but do not invent new behavior beyond documented flows | Maintains compatibility without over-guessing |
| Bulk scheduling transaction semantics | Preserve visible fail-fast error behavior and keep successful schedules durable | Closest safe interpretation of current behavior |
| Viewer implementation | Preserve the payload-inspection capability even if implementation differs | Support value matters more than the exact UI technology |
| Json result enum | Keep it if surrounding apps may consume it; otherwise document any removal explicitly | Avoid breaking hidden downstream dependencies |
