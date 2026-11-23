# CourtPass / RacketHub — Complete Engineering Development Plan & Checklist

> **Source of Truth:** Always follow [AGENTS.md](./AGENTS.md) and [CLAUDE.md](./CLAUDE.md) for business rules, financial math, concurrency invariants, and architectural constraints.  
> **Reference System Plan:** See [Court_Booking_Platform_System_Plan.md](./Court_Booking_Platform_System_Plan.md) for stakeholder context.  
> **Completion Gate (Every Milestone):** Run `php artisan test` and `vendor/bin/pint` (plus `npm run build` when frontend assets change) before marking any milestone complete.

---

## Quick Migration Guide (Moving to the New Project Folder)

When creating the new project at `C:\xampp\htdocs\court-booking`:

```bash
# 1. Create a fresh Laravel project in C:\xampp\htdocs\court-booking
cd C:\xampp\htdocs
composer create-project laravel/laravel court-booking
cd court-booking
composer require livewire/livewire

# 2. Copy these planning files from tournament-standings\docs into court-booking
copy C:\xampp\htdocs\tournament-standings\docs\AGENTS.md C:\xampp\htdocs\court-booking\AGENTS.md
copy C:\xampp\htdocs\tournament-standings\docs\CLAUDE.md C:\xampp\htdocs\court-booking\CLAUDE.md
copy C:\xampp\htdocs\tournament-standings\docs\DEV_PLAN.md C:\xampp\htdocs\court-booking\DEV_PLAN.md
mkdir C:\xampp\htdocs\court-booking\docs
copy C:\xampp\htdocs\tournament-standings\docs\Court_Booking_Platform_System_Plan.* C:\xampp\htdocs\court-booking\docs\
```

---

## Progress Overview

### Phase 1: MVP Pilot (Weeks 1 – 3 — Fastest Value)
- [ ] **Milestone 1.1:** Database Foundation, Enums, Models, Factories & Pilot Seeders
- [ ] **Milestone 1.2:** Auth, Multi-Venue Role Middleware (`court_owner` vs `super_admin`) & Audit Logger
- [ ] **Milestone 1.3:** Core Domain Services (`PartnerPricingCalculator`, `SlotAvailabilityService`, `BookingHoldService`, `WalkInBlockerService`) & Unit Tests
- [ ] **Milestone 1.4:** Portal 3 — Super Admin Setup (Venues, Courts, Rates, Partner Agreements & Manual GCash Verification)
- [ ] **Milestone 1.5:** Portal 2 — Court Owner Companion (Daily Agenda, **"Instant 1-Tap Block"**, QR/Code Check-In & Reschedule Tokens)
- [ ] **Milestone 1.6:** Portal 1 — Public Player Booking Web App (Venue Discovery, Live Slot Matrix, 10-Min Cart Hold, GCash Receipt Upload & Digital QR Pass)

### Phase 2: Automation & Payout Ledger (Weeks 4 – 6)
- [ ] **Milestone 2.1:** Automated 10-Minute Slot Hold Expiry Command & Scheduler
- [ ] **Milestone 2.2:** Automated Payment Gateway Integration (PayMongo / Xendit QRPh Webhooks)
- [ ] **Milestone 2.3:** Automated Weekly Court Owner Payout Settlement Ledger

### Phase 3: Community Open Play & Queuing Engine (Weeks 7 – 8)
- [ ] **Milestone 3.1:** *"Pamisa"*-Style Capped Open Play Sessions & Waitlist Engine
- [ ] **Milestone 3.2:** Live Digital Paddle Stack & On-Court Rotation Display

### Phase 4: Tournament Bridge & Multi-City Expansion (Month 3+)
- [ ] **Milestone 4.1:** Multi-Court Tournament Package Booking & Tournament Live Standings Bridge
- [ ] **Milestone 4.2:** Hardening, Security Audit & Laravel Cloud Deployment

---

## Milestone 1.1: Database Foundation, Enums, Models, Factories & Pilot Seeders

### Backend & Schema Tasks
- [ ] **Create Enums (`app/Enums/`):**
  - [ ] `Sport` (`pickleball`, `badminton` + labels and badge colors)
  - [ ] `UserRole` (`court_owner`, `super_admin`)
  - [ ] `VenueStatus` (`active`, `inactive`)
  - [ ] `PricingModel` (`markup`, `discount` + human-readable labels: `"Model A: Add-On Markup"`, `"Model B: Wholesale Discount (Ila i-less atoa)"`)
  - [ ] `BookingType` (`hourly`, `open_play`, `tournament`, `walk_in`)
  - [ ] `BookingStatus` (`held`, `awaiting_verification`, `confirmed`, `checked_in`, `cancelled`, `expired`)
  - [ ] `OpenPlayStatus` (`open`, `full`, `in_progress`, `completed`, `cancelled`)
  - [ ] `RegistrationStatus` (`registered`, `waitlist`, `checked_in`, `on_court`, `cancelled`)
  - [ ] `PayoutStatus` (`pending`, `disbursed`)
- [ ] **Create Migrations (`database/migrations/`):**
  - [ ] Create `venues` table (`id`, `name`, `slug` unique, `address`, `lat` nullable, `lng` nullable, `contact_phone`, `rules` text nullable, `operating_open_time` default `06:00:00`, `operating_close_time` default `22:00:00`, `status` default `active`, timestamps)
  - [ ] Add `role` and nullable `venue_id` (constrained to `venues`, `nullOnDelete`) to `users` table
  - [ ] Create `courts` table (`id`, `venue_id` constrained cascadeOnDelete, `name`, `sport`, `surface`, `is_active` default true, unique `[venue_id, name]`, timestamps)
  - [ ] Create `court_rates` table (`id`, `court_id` constrained cascadeOnDelete, `day_of_week` tinyint nullable, `start_time`, `end_time`, `hourly_rate` decimal(10,2), `is_peak` boolean default false, timestamps)
  - [ ] Create `partner_agreements` table (`id`, `venue_id` unique constrained cascadeOnDelete, `model` enum `markup|discount`, `markup_val` decimal(10,2) default 0, `discount_val` decimal(10,2) default 0, `bank_details` JSON nullable, timestamps)
  - [ ] Create `payout_settlements` table (`id`, `venue_id` constrained cascadeOnDelete, `date_from` date, `date_to` date, `total_court_earnings` decimal(12,2), `status` default `pending`, `reference_no` nullable, `disbursed_at` nullable, timestamps)
  - [ ] Create `bookings` table (`id`, `booking_code` unique, `venue_id` constrained cascadeOnDelete, `user_name`, `phone`, `booking_type`, `court_fee` decimal(10,2), `fee_addon` decimal(10,2), `owner_net_amount` decimal(10,2), `platform_fee_amount` decimal(10,2), `grand_total` decimal(10,2), `status`, `payment_proof_path` nullable, `payment_reference` nullable, `reschedule_token` unique nullable, `reschedule_expires_at` nullable, `hold_expires_at` nullable index, `payout_settlement_id` nullable constrained nullOnDelete, timestamps)
  - [ ] Create `booking_slots` table (`id`, `booking_id` constrained cascadeOnDelete, `court_id` constrained cascadeOnDelete, `date` date, `start_time` time, `end_time` time, `price` decimal(10,2), `active_lock_key` virtual/stored or composite index strategy so active bookings enforce uniqueness on `[court_id, date, start_time]`, timestamps)
  - [ ] Create `open_play_sessions` table (`id`, `venue_id` constrained cascadeOnDelete, `court_id` constrained cascadeOnDelete, `session_date` date, `start_time` time, `end_time` time, `max_players` unsignedSmallInteger default 16, `fee_per_player` decimal(10,2), `status` default `open`, timestamps)
  - [ ] Create `open_play_registrations` table (`id`, `session_id` constrained `open_play_sessions` cascadeOnDelete, `booking_id` nullable constrained nullOnDelete, `player_name`, `contact`, `queue_number` unsignedSmallInteger, `status` default `registered`, unique `[session_id, queue_number]`, timestamps)
  - [ ] Create `audit_logs` table (`id`, `user_id` nullable constrained nullOnDelete, `venue_id` nullable constrained nullOnDelete, `action`, `subject`, `old_values` JSON nullable, `new_values` JSON nullable, timestamps)
- [ ] **Create Models & Relationships (`app/Models/`):**
  - [ ] Configure `$fillable`, enum casts, and relationships for all 11 models (`Venue`, `Court`, `CourtRate`, `PartnerAgreement`, `Booking`, `BookingSlot`, `OpenPlaySession`, `OpenPlayRegistration`, `PayoutSettlement`, `AuditLog`, `User`)
- [ ] **Create Factories & Pilot Seeders (`database/factories/`, `database/seeders/`):**
  - [ ] Seed 1 `super_admin` user and 2 pilot `court_owner` users reading initial passwords from `.env` (`SUPER_ADMIN_INITIAL_PASSWORD`, `PILOT_OWNER_INITIAL_PASSWORD`)
  - [ ] Seed 2 pilot venues:
    - **Pilot Venue 1 (Model A — Add-On Markup):** 2 Pickleball courts (`₱300/hr` base + `₱30/hr` add-on = `₱330/hr` player checkout)
    - **Pilot Venue 2 (Model B — Wholesale Discount / "Ila i-less atoa"):** 2 Badminton courts (`₱300/hr` published SRP, `₱40/hr` partner discount = `₱260/hr` owner net, `₱300/hr` player checkout)

### Tests & Verification
- [ ] Write model & schema integrity tests asserting relationships, cascades, and unique constraints on `booking_slots`
- [ ] Write seeder test asserting both pilot venues, courts, rates, partner agreements, and initial role accounts are created cleanly
- [ ] **Done when:** `php artisan migrate:fresh --seed` and `php artisan test` pass with zero warnings.

---

## Milestone 1.2: Auth, Multi-Venue Role Middleware & Audit Logger

### Backend Tasks
- [ ] **Authentication:**
  - [ ] Build staff login and logout endpoints (`/login`, `POST /logout`) with rate limiting (`5` attempts per minute per email+IP)
  - [ ] Redirect authenticated users by role (`/admin` for `super_admin`, `/owner` for `court_owner`)
- [ ] **Authorization & Multi-Tenant Venue Isolation:**
  - [ ] Create `EnsureUserHasRole` middleware (`role:super_admin`, `role:court_owner,super_admin`)
  - [ ] Create `EnsureOwnerHasVenueAccess` helper/policy that guarantees a `court_owner` can only access records belonging to `auth()->user()->venue_id`
- [ ] **Audit Logging Service:**
  - [ ] Build `app/Services/AuditLogger.php` recording `user_id`, `venue_id`, `action`, `subject`, `old_values`, and `new_values`

### Frontend Tasks
- [ ] Responsive staff login page (`resources/views/auth/login.blade.php`)
- [ ] Shared staff layout (`resources/views/components/layouts/staff.blade.php`) with role badge, venue badge (for Court Owners), and quick switch to the Public Booking Page

### Tests & Verification
- [ ] Feature test: Guests are redirected to `/login` on `/owner` and `/admin`
- [ ] Feature test: `court_owner` receives `403 Forbidden` on `/admin` and cannot view or mutate another venue's data

---

## Milestone 1.3: Core Domain Services & Unit Tests

### Service Implementation (`app/Services/`)
- [ ] **`PartnerPricingCalculator`:**
  - [ ] Given a `Court`, `date`, `start_time`, and the venue's `PartnerAgreement`, resolve the applicable `CourtRate` (peak vs. off-peak) and return an immutable DTO/array:
    - `court_fee`
    - `fee_addon`
    - `owner_net_amount`
    - `platform_fee_amount`
    - `grand_total`
  - [ ] Verify **Model A (`markup`)**: `₱300` court fee + `₱30` add-on -> Player pays `₱330`, Court gets `₱300`, Platform gets `₱30`
  - [ ] Verify **Model B (`discount`)**: `₱300` court fee, `₱40` discount -> Player pays `₱300` (`₱0` add-on), Court gets `₱260`, Platform gets `₱40`
- [ ] **`SlotAvailabilityService`:**
  - [ ] Build daily slot matrix for a venue's courts from `operating_open_time` to `operating_close_time` in 1-hour increments
  - [ ] Classify each slot as `available`, `held` (`status = held` and `hold_expires_at > now()`), `booked` (`awaiting_verification`, `confirmed`, `checked_in`), or `walk_in_blocked` (`booking_type = walk_in`)
  - [ ] Treat any `held` booking whose `hold_expires_at <= now()` as expired/available
- [ ] **`BookingHoldService`:**
  - [ ] Inside `DB::transaction()`, lock target court rows (`lockForUpdate()`), release any expired `held` rows occupying those `(court_id, date, start_time)` coordinates, verify none of the requested slots are actively locked, snapshot pricing via `PartnerPricingCalculator`, and create the `Booking` (`status = held`, `hold_expires_at = now()->addMinutes(10)`) + `BookingSlot` rows
  - [ ] Throw a domain exception (`SlotAlreadyTakenException`) if any slot was taken concurrently
- [ ] **`WalkInBlockerService`:**
  - [ ] `blockSlot(User $actor, Court $court, string $date, string $startTime)`: atomically creates a `walk_in` confirmed booking (`0.00` fees) and logs to `AuditLogger`
  - [ ] `unblockSlot(User $actor, Booking $walkInBooking)`: cancels/removes a `walk_in` block if tapped by mistake and logs to `AuditLogger`
- [ ] **`PaymentVerificationService`:**
  - [ ] `submitManualProof(Booking $booking, UploadedFile $receipt, ?string $referenceNo)`: validates hold is still active, stores receipt image in private/public disk, transitions `held -> awaiting_verification`, clears `hold_expires_at`
  - [ ] `approvePayment(User $actor, Booking $booking)` & `rejectPayment(User $actor, Booking $booking, string $reason)`: transitions status, releases slots on rejection, and writes to `AuditLogger`
- [ ] **`RescheduleTokenService`:**
  - [ ] `issueToken(User $actor, Booking $booking, string $reason)`: cancels original slots, generates unique 30-day `reschedule_token`, logs to `AuditLogger`
  - [ ] `redeemToken(string $token, Venue $venue, array $newSlots)`: validates token belongs to the venue, is unused, and is within 30 days

### Tests & Verification
- [ ] Unit tests for `PartnerPricingCalculator` (Model A, Model B, multi-hour bookings, peak vs. off-peak rates, historical snapshot immutability after agreement change)
- [ ] Unit tests for `BookingHoldService` & `WalkInBlockerService` (concurrent slot collision rejection, reclaiming an expired 10-minute hold, preventing expiry once proof is uploaded)

---

## Milestone 1.4: Portal 3 — Super Admin (Manager View)

### Livewire Components (`app/Livewire/Admin/`)
- [ ] **`VenueManager` (`/admin/venues`):**
  - [ ] Create/edit partner venues, operating hours, contact info, and status (`active`/`inactive`)
  - [ ] Configure courts per venue (name, `pickleball` or `badminton`, surface type)
  - [ ] Configure hourly rates (off-peak vs. peak hours)
- [ ] **`PartnerAgreementForm` (`/admin/venues/{venue}/agreement`):**
  - [ ] Toggle between **Model A (Add-On Markup)** and **Model B (Wholesale Discount / "Ila i-less atoa")**
  - [ ] Live preview calculator showing: *"If court rate is ₱300/hr -> Player pays ₱X, Court Owner receives ₱Y, Our Platform keeps ₱Z"*
  - [ ] Bank / GCash disbursement details for the court owner
- [ ] **`PaymentVerificationQueue` (`/admin/payments`):**
  - [ ] List all `awaiting_verification` bookings with uploaded GCash receipt preview modal, reference number, amount expected, and 1-click **Approve** or **Reject**
- [ ] **`MasterFinancialDashboard` (`/admin`):**
  - [ ] KPI cards: Gross Booking Volume, Net Platform Revenue (our cut), Owed to Court Owners, Active Bookings Today
  - [ ] Filterable master transaction table and recent audit log feed

### Tests & Verification
- [ ] Feature test: Super Admin can onboard a venue, add courts/rates, switch partner agreement between Model A and Model B, and approve/reject an uploaded GCash receipt

---

## Milestone 1.5: Portal 2 — Court Owner Companion Portal

### Livewire Components (`app/Livewire/Owner/`)
- [ ] **`DailyCourtAgenda` (`/owner`):**
  - [ ] Mobile-first date switcher (Today, Tomorrow, 7-day strip) + court columns (`Court 1`, `Court 2`, ...)
  - [ ] `wire:poll.8s` live refresh
  - [ ] On every **Available** slot: prominent **"1-Tap Quick Block (Walk-In)"** button (`>= 44x44px` tap target) that locks the slot in < 2 seconds
  - [ ] On every **Walk-In Blocked** slot: **"Unblock"** button with confirmation
  - [ ] On every **Confirmed** online booking slot: customer name, phone, booking code, **"Check In"** button, and **"Issue Reschedule Token (Rain/Outage)"** action
- [ ] **`BookingCheckInScanner` (`/owner/check-in`):**
  - [ ] Search/verify by `booking_code` or camera QR scan to mark arriving players as `checked_in`
- [ ] **`OwnerEarningsLedger` (`/owner/earnings`):**
  - [ ] Transparent daily & weekly summary of confirmed online bookings, showing the venue's exact `owner_net_amount` per booking and payout disbursement history
- [ ] **`OwnerCourtSettings` (`/owner/settings`):**
  - [ ] Allow court owner to adjust their operating hours and court rates (logged to `audit_logs`)

### Tests & Verification
- [ ] Feature test: Court Owner can 1-tap block a walk-in slot (which immediately prevents public player booking for that slot) and unblock it
- [ ] Feature test: Court Owner can check in a confirmed booking and issue a 30-day reschedule token
- [ ] Security test: Court Owner A cannot block slots, check in bookings, or view earnings belonging to Court Owner B's venue

---

## Milestone 1.6: Portal 1 — Public Player Booking Web App

### Livewire Components (`app/Livewire/Public/`)
- [ ] **`VenueDirectory` (`/`):**
  - [ ] Mobile-first venue cards with sport filter (`All`, `Pickleball`, `Badminton`), search by name/location, surface badges, and starting hourly rate
- [ ] **`VenueSlotBooking` (`/venues/{venue:slug}`):**
  - [ ] Date picker strip + sport filter + interactive hourly court slot matrix (`Available` green, `Held` amber, `Booked` slate) with `wire:poll.8s`
  - [ ] Multi-slot selection cart (e.g., select Court 1 from 6:00 PM – 8:00 PM)
  - [ ] Transparent price breakdown in sticky bottom bar (under Model A shows `Court Fee + Convenience Fee`; under Model B shows `Standard Court Rate` with zero extra fee)
  - [ ] Option to apply a 30-day `reschedule_token` credit
  - [ ] Customer Name & Mobile Phone input -> clicking **"Proceed to Payment"** calls `BookingHoldService` and redirects to the checkout hold page
- [ ] **`BookingCheckoutHold` (`/bookings/{booking:booking_code}/checkout`):**
  - [ ] Live 10-minute countdown timer (`hold_expires_at`)
  - [ ] Platform GCash QR code / account number + exact `grand_total` to pay
  - [ ] Upload GCash receipt screenshot + optional reference number -> transitions booking to `awaiting_verification`
- [ ] **`DigitalBookingPass` (`/bookings/{booking:booking_code}`):**
  - [ ] Status banner (`Held — Complete Payment`, `Awaiting Verification`, `Confirmed — Ready to Play`, `Checked In`, or `Reschedule Token Issued`)
  - [ ] Scannable SVG QR Code encoding the verification URL / `booking_code`, venue name, court name, date, time slots, and player name (print/screenshot friendly)

### Tests & Verification
- [ ] Feature test: Full guest player flow — select slots -> 10-min hold created -> slot appears held on matrix -> upload GCash proof -> Super Admin approves -> Digital QR Booking Pass displays `Confirmed`
- [ ] Feature test: Two players attempting to hold the same slot at the same time — first succeeds, second receives a clean conflict validation message

---

## Milestone 2.1 – 2.3: Phase 2 — Automation & Payout Settlement Ledger

- [ ] **Automated Hold Expiry (`app/Console/Commands/ReleaseExpiredSlotHolds.php`):**
  - [ ] Scheduled every minute (`schedule->command('bookings:release-expired')->everyMinute()`) to mark `held` bookings where `hold_expires_at <= now()` as `expired`
- [ ] **PayMongo / Xendit QRPh Integration (`app/Services/PaymentGatewayService.php`):**
  - [ ] Generate dynamic QRPh / GCash checkout intent for a `held` booking
  - [ ] Signed webhook controller (`POST /webhooks/payments`) that atomically verifies signature, transitions `Booking` to `confirmed`, records `payment_reference`, and logs to `AuditLogger`
- [ ] **Automated Weekly Payout Ledger (`app/Services/PayoutSettlementService.php` & `/admin/payouts`):**
  - [ ] Group all unsettled `confirmed`/`checked_in` online bookings per venue for a selected date range
  - [ ] Sum `owner_net_amount` into a `PayoutSettlement` (`status = pending`) and link each included booking's `payout_settlement_id` inside a DB transaction
  - [ ] Allow Super Admin to record bank/GCash transfer `reference_no` and mark settlement as `disbursed` (visible immediately in the Court Owner's earnings tab)

---

## Milestone 3.1 – 3.2: Phase 3 — "Pamisa"-Style Open Play & Queuing Engine

- [ ] **`OpenPlayQueueService` (`app/Services/OpenPlayQueueService.php`):**
  - [ ] Create capped Open Play sessions (`max_players`, e.g. 16 players, `fee_per_player`, e.g. `₱150`)
  - [ ] Register player: inside `DB::transaction()`, lock the `OpenPlaySession` row, count active registrations, assign sequential `queue_number`, and set `status = registered` if `< max_players` or `status = waitlist` if full
  - [ ] Auto-promote first `waitlist` player when a `registered` slot is cancelled
- [ ] **Live Digital Paddle Stack (`/open-play/{session}` & `/owner/open-play/{session}`):**
  - [ ] Public live board (`wire:poll.5s`) showing **Now Playing on Court (4 players)**, **On Deck / Next Up (4 players)**, and **Paddle Queue Order**
  - [ ] Court Owner / Queue Master controls to rotate the next 4 players onto the court in one tap

---

## Milestone 4.1 – 4.2: Phase 4 — Tournament Bridge & Production Hardening

- [ ] **Tournament Package Booking & Standings Bridge:**
  - [ ] Allow organizers to book multi-court blocks under `booking_type = tournament`
  - [ ] Auto-generate or attach the live tournament URL linking to the **Tournament Live Standings** platform on the confirmed booking pass and venue schedule
- [ ] **Security Audit & Production Readiness:**
  - [ ] Run security audit across all Livewire actions (role checks, venue ownership checks, webhook signature verification, file upload MIME/size validation for GCash receipts)
  - [ ] Verify N+1 query prevention (`with(['courts', 'partnerAgreement'])`) and database indexes
  - [ ] Run full test suite (`php artisan test`), code formatter (`vendor/bin/pint`), and production build (`npm run build`)
