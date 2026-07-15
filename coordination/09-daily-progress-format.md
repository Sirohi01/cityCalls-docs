# Coordination 09 — Daily Progress Format

Lightweight, for a two-person team — not a heavyweight standup process.

## 1. Format

Each developer posts (in whatever shared channel they use — Slack/WhatsApp/etc., tool not prescribed here) at the end of a work session:

```
[Manish | Rohit] — {date}
Done: {what was completed, referencing module/phase}
In progress: {what's actively being worked on}
Blocked on: {anything waiting on the other developer or a decision, or "none"}
Next: {what's planned next}
```

## 2. Purpose

Surfaces blockers same-day rather than at the next integration checkpoint — specifically the "waiting on a contract" or "waiting on a UI spec" scenarios that [01-team-working-agreement.md](01-team-working-agreement.md) §2 flags as something to avoid discovering late.

## 3. Phase/Module Tracking

Progress against [22-phase-wise-development-plan.md](../22-phase-wise-development-plan.md) is tracked at the module level (not ticket-by-ticket) — a module is `Not Started` / `Contract Ready` / `Backend In Progress` / `UI In Progress` / `Integration Testing` / `Done` (per [23-definition-of-done.md](../23-definition-of-done.md)). Whoever updates this status does so the same day the state actually changes, not retroactively.

## 4. Weekly Check-in

A slightly longer weekly summary (what shipped this week, what's next week, any doc that needs revisiting) keeps the phase plan realistic — if a phase is consistently running behind its plan, that's a signal to revisit scope/sequencing in [22-phase-wise-development-plan.md](../22-phase-wise-development-plan.md) rather than silently letting the plan go stale.
