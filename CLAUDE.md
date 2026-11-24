# CLAUDE.md

@AGENTS.md
@DEV_PLAN.md

`AGENTS.md` is the single source of truth for **CourtPass / RacketHub**: scope, business rules, data model, slot waitlist & open-play queuing rules, and architecture. `DEV_PLAN.md` tracks the phase-by-phase implementation checklist. Everything below adds working guardrails. If the files ever disagree, `AGENTS.md` wins.

---

## How to Work in This Repo

1. Before writing code, restate the task in one or two lines and name the exact files you will touch.
2. Keep changes small and incremental: one feature, service, or component per step, paired with tests.
3. After editing PHP code, always run `php artisan test` and `vendor/bin/pint`, and report the actual terminal result.
4. Prefer project skills in `.agents/skills/` (`laravel-best-practices`, `livewire-development`, `tailwindcss-development`, `testing-best-practices`, `security-auditor`, `systematic-debugging`). Never use Bootstrap, React, or Vue skills.

---

## Non-Negotiable Guardrails

* **Zero payment or financial logic:** Do not add payment gateways, GCash receipt uploads, court rates, partner markup/discount calculators, reschedule credit tokens, or payout settlement tables. This platform is strictly for court booking and queuing.
* **Never allow double-booking:** Every instant slot booking, queue promotion, and walk-in block must go through `SlotBookingService` / `WalkInBlockerService` inside a database transaction (`DB::transaction()`) with row locking (`lockForUpdate()`) and the active `(court_id, date, start_time)` uniqueness guarantee (`active_lock_key`).
* **Enforce manual queue promotion & anti-hoarding limits:** Hourly bookings enforce `$venue->max_daily_hours_per_player` per phone number per day. When a confirmed booking is cancelled or marked no-show, the slot lock is cleared and the Court Owner manually triggers `promoteNextInQueue()` to confirm the next waitlisted player.
* **Never leak cross-venue data:** Every `court_owner` Livewire component and action must scope queries to `auth()->user()->venue_id` and authorize server-side inside every callable action method.
* **Never edit deployed migrations, commit `.env`, or hardcode initial passwords.**
* **Do not expand scope** (payments, native apps, player chat, multi-sport bracket rules) unless explicitly asked.

---

## Asking vs. Proceeding

Ask a clarifying question only when a wrong assumption would break a Business Rule in `AGENTS.md`, affect queue ordering/promotion behavior, or compromise slot concurrency. For minor UI/UX details, state your assumption in one line and proceed.
