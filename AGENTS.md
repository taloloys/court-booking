# AGENTS.md

Guidance for AI coding agents working on the **CourtPass / RacketHub** repository. Read this fully before making changes.

---

## Project Overview

**CourtPass / RacketHub** is a multi-venue court reservation, open-play queuing, and tournament venue booking platform for **Pickleball** and **Badminton**, inspired by the slot-allocation and fee-collection workflow of our church (*"Pamisa"*) reservation system.

* **Anyone with the public link (Players / Organizers)** can browse partner venues, view the live hourly court availability matrix, place a 10-minute temporary slot hold, pay via GCash / Maya / QRPh, receive a Digital Booking Pass with a verification QR code, or register for capped Open Play drop-in sessions (no player login required in Phase 1).
* **Court Owners (`court_owner`)** log into a fast, mobile-first companion portal scoped strictly to their venue to view their daily court agenda, tap **"Instant 1-Tap Block"** for physical walk-in players, scan player booking QR codes at check-in, issue 30-day reschedule tokens for rain/power interruptions, and track their daily earnings and weekly payouts.
* **Super Admins (`super_admin` — Management)** onboard partner venues, configure courts and peak/off-peak hourly rates, select each venue's revenue agreement (**Model A: Add-On Markup** vs. **Model B: Wholesale Discount / "Ila i-less atoa"**), verify manual GCash payments (Phase 1), generate weekly court owner payout settlements, and bridge Tournament Package bookings to the **Tournament Live Standings** app.

### Scope by Roadmap Phase

* **Phase 1 (MVP Pilot):** 1–2 pilot venues, dynamic hourly slot booking matrix, 10-minute cart slot hold, manual GCash receipt upload & verification, Digital QR Booking Pass, Court Owner 1-Tap Walk-in Blocker, and Super Admin venue/agreement setup.
* **Phase 2 (Automation):** PayMongo / Xendit (instant GCash/Maya/QRPh webhook confirmation), automated 10-minute hold release scheduler, and automated weekly court owner payout settlement ledger.
* **Phase 3 (Community Open Play):** *"Pamisa"*-style capped Open Play drop-in sessions (e.g., max 16 players), automated waitlists, and live digital paddle stack / on-court rotation board.
* **Phase 4 (Expansion & Tournament Bridge):** Multi-court Tournament Package bookings with automated provisioning/linking to the Tournament Live Standings app, plus multi-city venue filters.

**Out of scope (do not build unless explicitly asked):** native iOS/Android mobile apps, player social networks/chat, equipment e-commerce store, or modifying tournament bracket rules inside this repository.

---

## Tech Stack

* **Backend:** PHP 8.3+, Laravel 13
* **Database:** MySQL (`utf8mb4`)
* **Frontend:** Livewire + Blade, Tailwind CSS (via Vite)
* **Testing & Formatting:** Pest (or PHPUnit) for tests, Laravel Pint for formatting
* **Hosting:** Laravel Cloud (deploys from GitHub `main`)
* **Local Environment:** Windows + XAMPP, project at `C:\xampp\htdocs\court-booking`
* **Timezone:** `Asia/Manila`

Do not introduce Bootstrap, React, Vue, Inertia, or complex DDD/repository layers.

---

## Commands

```bash
composer install
npm install
php artisan migrate
php artisan db:seed
php artisan serve        # backend
npm run dev              # Vite, in a second terminal
php artisan test         # run the test suite
vendor/bin/pint          # format PHP code
npm run build            # production assets
```

Run `php artisan test` and `vendor/bin/pint` before considering any task done.

---

## Roles, Portals, and Permissions

Roles live in a `role` column on `users` (`court_owner`, `super_admin`), plus a nullable `venue_id` foreign key on `users` that binds a `court_owner` to their specific venue. Players book as public guests using their name, phone number, and a signed/unguessable `booking_code` voucher URL.

| Capability | Public Player | Court Owner (`court_owner`) | Super Admin (`super_admin`) |
| :--- | :---: | :---: | :---: |
| Browse venues, courts, rates & live slot matrix | Yes | Yes | Yes |
| Reserve hourly slots, upload GCash proof, view QR pass | Yes | Yes | Yes |
| Register for Open Play sessions & view live paddle queue | Yes | Yes | Yes |
| View daily court agenda & scan booking QR for check-in | No | Own Venue Only | All Venues |
| **1-Tap Quick Block** / unblock slots for walk-in players | No | Own Venue Only | All Venues |
| Issue 30-day Reschedule Token for rain/power outage | No | Own Venue Only | All Venues |
| Adjust court operating hours & peak/off-peak rates | No | Own Venue Only | All Venues |
| View venue earnings & payout settlement history | No | Own Venue Only | All Venues |
| Verify manual GCash payment uploads | No | Optional (if enabled) | Yes |
| Onboard venues & configure Partner Agreements (Model A / B) | No | No | Yes |
| Generate & mark weekly Payout Settlements as disbursed | No | No | Yes |
| View global Audit Logs & master platform revenue ledger | No | No | Yes |

**Authorization & Multi-Tenant Isolation Rules:**
1. Authorization is enforced server-side inside every Livewire action (Gate/Policy or explicit role + venue check), never only via route middleware or hidden buttons.
2. Every `court_owner` query and action **must** verify `$venue->id === auth()->user()->venue_id`. A Court Owner must never read or mutate another venue's courts, bookings, or payouts.

---

## Data Model

1. **`venues`**: `id`, `name`, `slug` (unique), `address`, `lat` (nullable), `lng` (nullable), `contact_phone`, `rules` (nullable), `operating_open_time` (default `06:00`), `operating_close_time` (default `22:00`), `status` (`active` | `inactive`).
2. **`courts`**: `id`, `venue_id`, `name` (e.g. `Court 1`), `sport` (`pickleball` | `badminton`), `surface` (e.g. `taraflex`, `wood`, `acrylic`, `concrete`), `is_active` (boolean).
3. **`court_rates`**: `id`, `court_id`, `day_of_week` (`0`–`6`, or nullable for all days), `start_time`, `end_time`, `hourly_rate` (decimal), `is_peak` (boolean).
4. **`partner_agreements`**: `id`, `venue_id` (unique), `model` (`markup` | `discount`), `markup_val` (decimal, per hour add-on), `discount_val` (decimal, per hour wholesale reduction), `bank_details` (JSON/text for payout disbursement).
5. **`bookings`**: `id`, `booking_code` (unique, e.g. `CP-20261123-X8F2`), `venue_id`, `user_name`, `phone`, `booking_type` (`hourly` | `open_play` | `tournament` | `walk_in`), `court_fee` (snapshotted base court price), `fee_addon` (snapshotted player-facing markup; `0` under Model B), `owner_net_amount` (snapshotted amount owed to court owner), `platform_fee_amount` (snapshotted platform profit), `grand_total` (`court_fee + fee_addon`), `status` (`held` | `awaiting_verification` | `confirmed` | `checked_in` | `cancelled` | `expired`), `payment_proof_path` (nullable), `payment_reference` (nullable), `reschedule_token` (nullable, unique), `reschedule_expires_at` (nullable), `hold_expires_at` (nullable timestamp), `payout_settlement_id` (nullable FK).
6. **`booking_slots`**: `id`, `booking_id`, `court_id`, `date` (date), `start_time` (time), `end_time` (time), `price` (decimal), `is_active_lock` (boolean/virtual or managed on active booking statuses).
   * **Concurrency Invariant:** Active slots (`held`, `awaiting_verification`, `confirmed`, `checked_in`) must never overlap for the same `(court_id, date, start_time)`.
7. **`open_play_sessions`**: `id`, `venue_id`, `court_id`, `session_date`, `start_time`, `end_time`, `max_players` (int, e.g. `16`), `fee_per_player` (decimal), `status` (`open` | `full` | `in_progress` | `completed` | `cancelled`).
8. **`open_play_registrations`**: `id`, `session_id`, `booking_id` (nullable), `player_name`, `contact`, `queue_number` (int), `status` (`registered` | `waitlist` | `checked_in` | `on_court` | `cancelled`).
9. **`payout_settlements`**: `id`, `venue_id`, `date_from`, `date_to`, `total_court_earnings` (decimal), `status` (`pending` | `disbursed`), `reference_no` (nullable), `disbursed_at` (nullable timestamp).
10. **`audit_logs`**: `id`, `user_id` (nullable), `venue_id` (nullable), `action`, `subject`, `old_values` (JSON nullable), `new_values` (JSON nullable), `created_at`.
11. **`users`**: standard Laravel table plus `role` (`court_owner` | `super_admin`) and `venue_id` (nullable FK to `venues`).

---

## Business Rules (Do Not Change Without Asking)

1. **Zero Double-Booking Invariant:**
   * All slot reservations and walk-in blocks must run inside `DB::transaction()` using row locks (`lockForUpdate()`) and a database-level uniqueness constraint so two concurrent requests can never reserve the same `(court_id, date, start_time)`.
2. **10-Minute Temporary Cart Hold:**
   * When a player initiates checkout for one or more 1-hour slots, create the booking with `status = held` and `hold_expires_at = now()->addMinutes(10)`.
   * While `status = held` and `hold_expires_at > now()`, the slot shows as **Reserved / Held** on the public matrix.
   * If `hold_expires_at <= now()` and no payment proof has been submitted, the hold is treated as expired and can be released/reclaimed immediately.
   * Once a player uploads a GCash receipt (Phase 1: `status = awaiting_verification`) or PayMongo webhook confirms payment (Phase 2: `status = confirmed`), `hold_expires_at` is cleared and the slot remains locked.
3. **Revenue & Monetization Engine ("Add-on or Less"):**
   * Financial math is calculated by `PartnerPricingCalculator` per 1-hour slot and **snapshotted** onto `bookings` at creation time:
     * **Model A (`markup` — The "Add-On"):**
       * `court_fee` = sum of court's hourly rates (e.g. `₱300/hr`)
       * `fee_addon` = `markup_val * slot_count` (e.g. `₱30/hr`)
       * `grand_total` = `court_fee + fee_addon` (`₱330/hr`)
       * `owner_net_amount` = `court_fee` (`₱300/hr`)
       * `platform_fee_amount` = `fee_addon` (`₱30/hr`)
     * **Model B (`discount` — "Ila i-less atoa"):**
       * `court_fee` = sum of court's published hourly rates (e.g. `₱300/hr`)
       * `fee_addon` = `0` *(zero markup shown to player)*
       * `grand_total` = `court_fee` (`₱300/hr`)
       * `owner_net_amount` = `court_fee - (discount_val * slot_count)` (e.g. `₱260/hr`)
       * `platform_fee_amount` = `discount_val * slot_count` (e.g. `₱40/hr`)
   * Updating a venue's `partner_agreements` only affects new bookings, never historical bookings.
4. **Court Owner "Instant 1-Tap Block" for Walk-ins:**
   * Caretakers on-site can block any open slot on their venue's daily agenda in one tap.
   * A walk-in block creates a `bookings` row with `booking_type = walk_in`, `status = confirmed`, and `0.00` platform fee/payout values, locking the slot in `booking_slots` immediately. Caretakers can also unblock a `walk_in` slot if tapped by mistake.
5. **Open Play Capped Sessions ("Pamisa" Style):**
   * Each Open Play session has a strict `max_players` cap (e.g. 16 players).
   * Confirmed registrations up to `max_players` get `status = registered`. Registrations after the cap is reached receive `status = waitlist` ordered by `queue_number`.
   * If a registered player cancels, the first `waitlist` player can be promoted into `registered`.
6. **Weather / Power Outage Reschedule Token:**
   * No-shows are non-refundable, so full payment is collected at booking.
   * For certified rainouts or power interruptions, a Court Owner or Super Admin can click **"Issue Reschedule Token"** on a confirmed booking. This cancels and releases the original slots, generates a unique `reschedule_token` valid for 30 days (`now()->addDays(30)`), and allows the player to apply that credit to a new slot at the same venue.
7. **Payout Settlements:**
   * Only `confirmed` and `checked_in` online bookings (`booking_type != walk_in`) that have not yet been linked to a `payout_settlement_id` are included when generating a venue's payout settlement for a date range.
   * Sum of `owner_net_amount` becomes `total_court_earnings`.
8. **Tournament Standings Bridge:**
   * When a `tournament` package booking is confirmed, the system can store or generate a deep link / reference URL to the client's **Tournament Live Standings** application.
9. **Audit Trail:**
   * Every payment verification/rejection, walk-in block/unblock, reschedule token issuance, partner agreement update, rate change, and payout disbursement is recorded in `audit_logs`.

---

## Architecture

```text
app/
  Models/            Venue, Court, CourtRate, PartnerAgreement, Booking, BookingSlot,
                     OpenPlaySession, OpenPlayRegistration, PayoutSettlement, AuditLog, User
  Enums/             Sport, UserRole, PricingModel, BookingType, BookingStatus,
                     VenueStatus, OpenPlayStatus, RegistrationStatus, PayoutStatus
  Services/          SlotAvailabilityService, PartnerPricingCalculator, BookingHoldService,
                     WalkInBlockerService, PaymentVerificationService, RescheduleTokenService,
                     OpenPlayQueueService, PayoutSettlementService, AuditLogger
  Livewire/          Public/, Owner/, Admin/
  Http/Middleware/   EnsureUserHasRole, EnsureOwnerHasVenueAccess
```

* Keep all business logic in `app/Services`, never inside Livewire components or Blade views.
* `PartnerPricingCalculator`, `BookingHoldService`, and `SlotAvailabilityService` are core domain services and must be thoroughly unit tested (including race conditions, hold expiration, Model A vs. Model B math, and walk-in blocks).

---

## Frontend and UX

* **Mobile-First:** Both players booking on phones and court caretakers tapping **"1-Tap Block"** on-site use mobile screens. Every interactive slot button must have a minimum `44x44px` touch target.
* **Portal 1 (Player Matrix):** Visual hourly grid per court (`Green` = Available, `Amber` = Temporarily Held, `Slate/Gray` = Booked). Sticky bottom checkout bar showing selected slots, live countdown timer during hold, and clear total price.
* **Portal 2 (Court Owner Companion):** Zero clutter. Defaults directly to **Today's Court Grid** with 1-tap `Block Walk-In` on empty slots and 1-tap `Check In` / `Verify QR` on booked slots.
* **Live Updates:** Use `wire:poll.8s` on the public slot matrix and owner daily agenda so newly held or walk-in-blocked slots update automatically without page refresh.
* **Styling:** Tailwind CSS utilities only. High-contrast athletic palette (Deep Slate navy headers, Court Emerald accents for Pickleball/Badminton availability, Amber for holds/alerts).

---

## Working Agreement

1. Follow the build order in `DEV_PLAN.md`: migrations & enums -> models & seeders -> core pricing & slot-locking services with unit tests -> auth & tenant middleware -> Super Admin portal -> Court Owner companion -> Public Player booking flow -> Open Play & Payouts.
2. If a request conflicts with the Business Rules above, stop and ask instead of working around it.
3. Never commit `.env` or hardcoded credentials. Seed initial accounts from `.env` variables (`SUPER_ADMIN_INITIAL_PASSWORD`, `PILOT_OWNER_INITIAL_PASSWORD`).
4. Run `php artisan test` and `vendor/bin/pint` after every implementation phase.
