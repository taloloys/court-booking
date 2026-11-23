# CLAUDE.md

@AGENTS.md
@DEV_PLAN.md

`AGENTS.md` is the single source of truth for **CourtPass / RacketHub**: scope, business rules, data model, revenue math, and architecture. `DEV_PLAN.md` tracks the phase-by-phase implementation checklist. Everything below adds working guardrails. If the files ever disagree, `AGENTS.md` wins.

---

## How to Work in This Repo

1. Before writing code, restate the task in one or two lines and name the exact files you will touch.
2. Keep changes small and incremental: one feature, service, or component per step, paired with tests.
3. After editing PHP code, always run `php artisan test` and `vendor/bin/pint`, and report the actual terminal result.
4. Prefer project skills in `.agents/skills/` (`laravel-best-practices`, `livewire-development`, `tailwindcss-development`, `testing-best-practices`, `security-auditor`, `systematic-debugging`). Never use Bootstrap, React, or Vue skills.

---

## Non-Negotiable Guardrails

* **Never allow double-booking:** Every slot reservation and walk-in block must go through `BookingHoldService` / `WalkInBlockerService` inside a database transaction (`DB::transaction()`) with row locking and the `(court_id, date, start_time)` uniqueness guarantee.
* **Never compute historical payouts from live `partner_agreements`:** Always snapshot `court_fee`, `fee_addon`, `owner_net_amount`, `platform_fee_amount`, and `grand_total` on the `bookings` row when the booking is created.
* **Never leak cross-venue data:** Every `court_owner` Livewire component and action must scope queries to `auth()->user()->venue_id` and authorize server-side inside every callable action method.
* **Never auto-expire a booking after payment proof is uploaded:** Only `status = held` bookings with `hold_expires_at <= now()` expire automatically. Once `status = awaiting_verification` or `confirmed`, the slot stays locked until verified, completed, or explicitly cancelled.
* **Never edit deployed migrations, commit `.env`, or hardcode initial passwords.**
* **Do not expand scope** (native apps, player chat, multi-sport bracket rules) unless explicitly asked.

---

## Asking vs. Proceeding

Ask a clarifying question only when a wrong assumption would break a Business Rule in `AGENTS.md`, affect payment/payout math, or compromise slot concurrency. For minor UI/UX details, state your assumption in one line and proceed.
