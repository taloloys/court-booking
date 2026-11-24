# CourtPass / RacketHub — Complete Engineering Development Plan & Checklist

> **Source of Truth:** Always follow [AGENTS.md](./AGENTS.md) and [CLAUDE.md](./CLAUDE.md) for business rules, slot concurrency invariants, queuing mechanics, and architectural constraints.  
> **Reference System Plan:** See [docs/Court_Booking_Platform_System_Plan.md](./docs/Court_Booking_Platform_System_Plan.md) for stakeholder context.  
> **UI/UX & Color Theme Specification:** Always follow [docs/rally_color_theme_rationale.md](./docs/rally_color_theme_rationale.md) (**RALLY** palette: Deep Forest Green `#173F32`, Court Green `#24764E`, Electric Lime `#C8F169`, Warm Ivory `#F4F1E7`, Soft White `#FFFFFF`, Charcoal `#252B28`).  
> **Completion Gate (Every Milestone):** Run `php artisan test` and `vendor/bin/pint` (plus `npm run build` when frontend assets change) before marking any milestone complete.

---

## Quick Migration Guide (Moving to the Project Folder)

When setting up the project at `C:\xampp\htdocs\court-booking`:

```bash
# 1. Create a fresh Laravel project in C:\xampp\htdocs\court-booking
cd C:\xampp\htdocs
composer create-project laravel/laravel court-booking
cd court-booking
composer require livewire/livewire

# 2. Verify test suite & formatting
php artisan test
vendor/bin/pint
```

---

## Progress Overview

### Phase 1: MVP Pilot — Core Court Booking & Slot Waitlist Queue (Weeks 1 – 3)
- [ ] **Milestone 1.1:** Database Foundation, Enums, Models, Factories & Pilot Seeders (Zero-Payment Schema)
- [ ] **Milestone 1.2:** Auth, Multi-Venue Role Middleware (`court_owner` vs `super_admin`) & Audit Logger
- [ ] **Milestone 1.3:** Core Domain Services (`SlotAvailabilityService`, `SlotBookingService`, `WalkInBlockerService`) & Unit Tests
- [ ] **Milestone 1.4:** Portal 3 — Super Admin Setup (Venues, Courts, Global Bookings/Queue Monitor & Audit Feed)
- [ ] **Milestone 1.5:** Portal 2 — Court Owner Companion (Daily Agenda, **"Instant 1-Tap Block"**, **Slot Waitlist Queue Promotion** & QR/Code Check-In)
- [ ] **Milestone 1.6:** Portal 1 — Public Player Booking Web App (Venue Discovery, Live Slot Matrix, Instant Booking, **"Join Slot Waitlist Queue"** & Digital QR Pass with Self-Cancel)

### Phase 2: Community Open Play & Paddle Stack Queuing Engine (Weeks 4 – 5)
- [ ] **Milestone 2.1:** *"Pamisa"*-Style Capped Open Play Sessions & Waitlist Engine (`OpenPlayQueueService`)
- [ ] **Milestone 2.2:** Live Digital Paddle Stack & 4-Player On-Court Rotation Display

### Phase 3: Tournament Bridge & Multi-City Expansion (Week 6+)
- [ ] **Milestone 3.1:** Multi-Court Tournament Block Booking & Tournament Live Standings Bridge
- [ ] **Milestone 3.2:** Hardening, Security Audit & Laravel Cloud Deployment

---

## Milestone 1.1: Database Foundation, Enums, Models, Factories & Pilot Seeders

### Backend & Schema Tasks
- [ ] **Create Enums (`app/Enums/`):**
  - [ ] `Sport` (`pickleball`, `badminton` + labels and badge colors)
  - [ ] `UserRole` (`court_owner`, `super_admin`)
  - [ ] `VenueStatus` (`active`, `inactive`)
  - [ ] `BookingType` (`hourly`, `open_play`, `tournament`, `walk_in`)
  - [ ] `BookingStatus` (`confirmed`, `waitlisted`, `checked_in`, `cancelled`, `no_show`)
  - [ ] `OpenPlayStatus` (`open`, `full`, `in_progress`, `completed`, `cancelled`)
  - [ ] `RegistrationStatus` (`registered`, `waitlist`, `checked_in`, `on_court`, `cancelled`)
- [ ] **Create Migrations (`database/migrations/`):**
  - [ ] Create `venues` table (`id`, `name`, `slug` unique, `address`, `lat` nullable, `lng` nullable, `contact_phone`, `rules` text nullable, `operating_open_time` default `06:00:00`, `operating_close_time` default `22:00:00`, `max_daily_hours_per_player` unsignedTinyInteger default `2`, `status` default `active`, timestamps)
  - [ ] Add `role` and nullable `venue_id` (constrained to `venues`, `nullOnDelete`) to `users` table
  - [ ] Create `courts` table (`id`, `venue_id` constrained cascadeOnDelete, `name`, `sport`, `surface`, `is_active` default true, unique `[venue_id, name]`, timestamps)
  - [ ] Create `bookings` table (`id`, `booking_code` unique, `venue_id` constrained cascadeOnDelete, `user_name`, `phone`, `booking_type`, `status` default `confirmed`, `queue_position` unsignedSmallInteger nullable, `tournament_standings_url` nullable, `cancelled_reason` nullable, timestamps)
  - [ ] Create `booking_slots` table (`id`, `booking_id` constrained cascadeOnDelete, `court_id` constrained cascadeOnDelete, `date` date, `start_time` time, `end_time` time, `active_lock_key` nullable unique index strategy so at most one active primary booking (`confirmed` or `checked_in`) can hold `[court_id, date, start_time]`, plus composite index on `[court_id, date, start_time]`, timestamps)
  - [ ] Create `open_play_sessions` table (`id`, `venue_id` constrained cascadeOnDelete, `court_id` constrained cascadeOnDelete, `session_date` date, `start_time` time, `end_time` time, `max_players` unsignedSmallInteger default 16, `status` default `open`, timestamps)
  - [ ] Create `open_play_registrations` table (`id`, `session_id` constrained `open_play_sessions` cascadeOnDelete, `booking_id` nullable constrained nullOnDelete, `player_name`, `contact`, `queue_number` unsignedSmallInteger, `status` default `registered`, unique `[session_id, queue_number]`, timestamps)
  - [ ] Create `audit_logs` table (`id`, `user_id` nullable constrained nullOnDelete, `venue_id` nullable constrained nullOnDelete, `action`, `subject`, `old_values` JSON nullable, `new_values` JSON nullable, timestamps)
- [ ] **Create Models & Relationships (`app/Models/`):**
  - [ ] Configure `$fillable`, enum casts, and relationships for all 8 models (`Venue`, `Court`, `Booking`, `BookingSlot`, `OpenPlaySession`, `OpenPlayRegistration`, `AuditLog`, `User`)
- [ ] **Create Factories & Pilot Seeders (`database/factories/`, `database/seeders/`):**
  - [ ] Seed 1 `super_admin` user and 2 pilot `court_owner` users reading initial passwords from `.env` (`SUPER_ADMIN_INITIAL_PASSWORD`, `PILOT_OWNER_INITIAL_PASSWORD`)
  - [ ] Seed 2 pilot venues:
    - **Pilot Venue 1 (Metro Pickleball Club):** 2 Pickleball courts (`Court 1`, `Court 2` — `taraflex`, `06:00` to `22:00`, max 2 hrs/day per player)
    - **Pilot Venue 2 (Smash Zone Badminton Center):** 2 Badminton courts (`Court 1`, `Court 2` — `wood`, `06:00` to `22:00`, max 2 hrs/day per player)

### Tests & Verification
- [ ] Write model & schema integrity tests asserting relationships, cascades, and unique active-lock constraints on `booking_slots` (allowing 1 active primary booking + multiple waitlisted queue entries per slot)
- [ ] Write seeder test asserting both pilot venues, courts, and initial role accounts are created cleanly
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

### Frontend Tasks (RALLY Design System Foundation)
- [ ] Configure **RALLY** Tailwind CSS theme tokens in `resources/css/app.css` per [docs/rally_color_theme_rationale.md](./docs/rally_color_theme_rationale.md):
  - `rally-forest` (`#173F32` — Deep Forest Green: nav, footer, hero headers, dark text on lime)
  - `rally-court` (`#24764E` — Court Green: secondary buttons, active links, `"Available"` status)
  - `rally-lime` (`#C8F169` — Electric Lime: primary CTAs, selected date/time slots, badges — always with `#173F32`/`#252B28` dark text)
  - `rally-ivory` (`#F4F1E7` — Warm Ivory: main page canvas)
  - `rally-surface` (`#FFFFFF` — Soft White: court cards, forms, panels, drawers)
  - `rally-charcoal` (`#252B28` — Charcoal: readable body text & form labels)
- [ ] Public guest layout (`resources/views/components/layouts/app.blade.php`) with Deep Forest Green (`#173F32`) navbar/footer and Warm Ivory (`#F4F1E7`) page canvas
- [ ] Responsive staff login page (`resources/views/auth/login.blade.php`) styled with the RALLY palette
- [ ] Shared staff layout (`resources/views/components/layouts/staff.blade.php`) with Deep Forest Green header, Electric Lime active badges, role/venue badge, and quick switch to the Public Booking Page

### Tests & Verification
- [ ] Feature test: Guests are redirected to `/login` on `/owner` and `/admin`
- [ ] Feature test: `court_owner` receives `403 Forbidden` on `/admin` and cannot view or mutate another venue's data

---

## Milestone 1.3: Core Domain Services & Unit Tests

### Service Implementation (`app/Services/`)
- [ ] **`SlotAvailabilityService`:**
  - [ ] Build daily slot matrix for a venue's courts from `operating_open_time` to `operating_close_time` in 1-hour increments
  - [ ] Classify each slot as:
    - `available` (no active `confirmed` or `checked_in` booking)
    - `booked` (active online booking with `status = confirmed` or `checked_in`)
    - `walk_in_blocked` (`booking_type = walk_in` and `status = confirmed`)
  - [ ] Include waitlist metadata on each slot (`waitlist_count`, ordered list of `waitlisted` entries by `queue_position`) so both players and court owners can see the current queue depth
- [ ] **`SlotBookingService`:**
  - [ ] **`bookAvailableSlots(Venue $venue, string $userName, string $phone, array $requestedSlots)`:**
    - Inside `DB::transaction()`, lock target court rows (`lockForUpdate()`).
    - Enforce daily anti-hoarding cap: check total active/waitlisted hours booked by `$phone` at `$venue` on the target date against `$venue->max_daily_hours_per_player` (e.g., max 2 hours/day), throwing `DailyBookingLimitExceededException` if exceeded.
    - Verify none of the requested `(court_id, date, start_time)` coordinates currently have an active lock (`confirmed` or `checked_in`), throwing `SlotAlreadyTakenException` if taken.
    - Create the `Booking` (`status = confirmed`, `booking_type = hourly`, unique `booking_code`) and `BookingSlot` rows with `active_lock_key` set.
  - [ ] **`joinSlotWaitlist(Court $court, string $date, string $startTime, string $userName, string $phone)`:**
    - Inside `DB::transaction()`, lock the court row (`lockForUpdate()`).
    - Enforce daily anti-hoarding cap and prevent the same `$phone` from joining the same slot's waitlist twice.
    - Verify the slot currently has an active booking (if the slot is actually open, either book directly or prompt).
    - Assign the next sequential `queue_position` (`1, 2, 3...`) for that `(court_id, date, start_time)` and create a `Booking` (`status = waitlisted`, `queue_position = $next`) + `BookingSlot` (`active_lock_key = null`).
  - [ ] **`cancelOrMarkNoShow(Booking $booking, ?User $actor = null, BookingStatus $newStatus = BookingStatus::Cancelled, ?string $reason = null)`:**
    - Inside `DB::transaction()`, transition the booking to `cancelled` or `no_show`, clear `active_lock_key` on its `BookingSlot` rows so the slot is released, and log to `AuditLogger`.
    - Note: Does **not** auto-promote the waitlist; the slot becomes open for the Court Owner to manually promote the next queued player (or for a new booking if the queue is empty).
  - [ ] **`promoteNextInQueue(User $actor, Court $court, string $date, string $startTime, ?Booking $specificWaitlistBooking = null)`:**
    - Inside `DB::transaction()`, lock the court row (`lockForUpdate()`).
    - Verify no active primary booking (`confirmed` or `checked_in`) currently holds `(court_id, date, start_time)`.
    - Select the lowest `queue_position` `waitlisted` booking for that `(court_id, date, start_time)` (or `$specificWaitlistBooking` if specified by the owner).
    - Transition it to `status = confirmed`, set `active_lock_key` on its `BookingSlot` row, clear `queue_position`, and log the promotion to `AuditLogger`.
- [ ] **`WalkInBlockerService`:**
  - [ ] `blockSlot(User $actor, Court $court, string $date, string $startTime)`: atomically creates a `walk_in` confirmed booking locking `(court_id, date, start_time)` and logs to `AuditLogger`
  - [ ] `unblockSlot(User $actor, Booking $walkInBooking)`: cancels/removes a `walk_in` block if tapped by mistake, clears `active_lock_key`, and logs to `AuditLogger`

### Tests & Verification
- [ ] Unit tests for `SlotBookingService` (instant confirmation, concurrent slot collision rejection, enforcing daily max hours per phone number, joining a slot waitlist with sequential `queue_position`, self-cancellation releasing the slot lock, and manual owner promotion of the next waitlisted player into `confirmed`)
- [ ] Unit tests for `WalkInBlockerService` (atomic walk-in block, preventing public confirmation while blocked, and unblocking)

---

## Milestone 1.4: Portal 3 — Super Admin (Manager View)

### Livewire Components (`app/Livewire/Admin/`)
- [ ] **`VenueManager` (`/admin/venues`):**
  - [ ] Create/edit partner venues, operating hours (`operating_open_time`, `operating_close_time`), `max_daily_hours_per_player` anti-hoarding limit, contact info, and status (`active`/`inactive`)
  - [ ] Assign/manage `court_owner` accounts per venue
  - [ ] Configure courts per venue (name, `pickleball` or `badminton`, surface type, active status)
- [ ] **`AdminDashboard` (`/admin`):**
  - [ ] Operational KPI cards: Active Venues & Courts, Confirmed Bookings Today, Players Currently in Slot Waitlist Queues, Checked-In Sessions Today
  - [ ] Filterable master bookings & waitlist queue monitor across all venues (filter by venue, date, court, status)
  - [ ] Recent `audit_logs` feed (walk-in blocks, queue promotions, cancellations, no-shows, venue config updates)

### Tests & Verification
- [ ] Feature test: Super Admin can onboard a venue, configure daily slot limits, add courts, view global bookings/queues, and inspect audit logs

---

## Milestone 1.5: Portal 2 — Court Owner Companion Portal

### Livewire Components (`app/Livewire/Owner/`)
- [ ] **`DailyCourtAgenda` (`/owner`):**
  - [ ] Mobile-first date switcher (Today, Tomorrow, 7-day strip) + court columns (`Court 1`, `Court 2`, ...)
  - [ ] `wire:poll.8s` live refresh
  - [ ] On every **Available** slot (with empty waitlist): prominent **"1-Tap Quick Block (Walk-In)"** button (`>= 44x44px` tap target) that locks the slot in < 2 seconds
  - [ ] On every **Available** slot that has a **Waitlist Queue** (e.g., after a cancellation/no-show): prominent **"Promote Next in Queue (#1: Name - Phone)"** button + option to inspect the full queue list or 1-tap block for walk-in
  - [ ] On every **Walk-In Blocked** slot: **"Unblock"** button with confirmation
  - [ ] On every **Confirmed** online booking slot: customer name, phone, booking code, waitlist badge (`Queue: N waiting`), **"Check In"** button, and **"Mark No-Show / Cancel"** action (which frees the slot so the owner can immediately tap **"Promote Next in Queue"**)
- [ ] **`BookingCheckInScanner` (`/owner/check-in`):**
  - [ ] Search/verify by `booking_code` or camera QR scan to mark arriving players as `checked_in`
- [ ] **`OwnerCourtSettings` (`/owner/settings`):**
  - [ ] Allow court owner to adjust their venue's operating hours, `max_daily_hours_per_player` limit, and court active status (logged to `audit_logs`)

### Tests & Verification
- [ ] Feature test: Court Owner can 1-tap block a walk-in slot (which immediately prevents public instant confirmation for that slot) and unblock it
- [ ] Feature test: Court Owner can check in a confirmed booking, mark a booking as No-Show/Cancelled, and manually promote the #1 waitlisted player in the queue to `confirmed`
- [ ] Security test: Court Owner A cannot block slots, promote queues, check in bookings, or edit settings belonging to Court Owner B's venue

---

## Milestone 1.6: Portal 1 — Public Player Booking Web App

### Livewire Components (`app/Livewire/Public/`)
- [ ] **`VenueDirectory` (`/`):**
  - [ ] Mobile-first venue cards with sport filter (`All`, `Pickleball`, `Badminton`), search by name/location, surface badges, court counts, and operating hours
- [ ] **`VenueSlotBooking` (`/venues/{venue:slug}`):**
  - [ ] Date picker strip + sport filter + interactive hourly court slot matrix on Soft White (`#FFFFFF`) court cards over a Warm Ivory (`#F4F1E7`) canvas with `wire:poll.8s`:
    - **Available:** Court Green (`#24764E`) indicator paired with explicit `"Available"` label
    - **Selected:** Electric Lime (`#C8F169`) background with Deep Forest Green (`#173F32`) text and `"Selected"` label
    - **Booked (Join Waitlist Queue):** Amber indicator paired with `"Booked · Join Queue (#N)"` label
    - **Unavailable / Walk-In Blocked:** Muted gray paired with `"Unavailable"` / `"Walk-In Blocked"` label
  - [ ] **Mode A — Instant Slot Booking (Available Slots):**
    - Select one or more available 1-hour slots (up to `venue->max_daily_hours_per_player`)
    - Enter Player Name & Mobile Phone in the sticky bottom bar -> clicking the Electric Lime (`#C8F169`) **"Confirm Court Booking"** CTA calls `SlotBookingService::bookAvailableSlots()` and redirects directly to the confirmed `DigitalBookingPass`
  - [ ] **Mode B — Join Slot Waitlist Queue (Booked Slots):**
    - Tapping an already-booked slot opens a quick **"Join Waitlist Queue for [Court] at [Time]"** drawer showing current queue length (`#N in line`)
    - Enter Player Name & Mobile Phone -> clicking **"Join Waitlist Queue"** calls `SlotBookingService::joinSlotWaitlist()` and redirects to the `DigitalBookingPass` showing their live queue rank
- [ ] **`DigitalBookingPass` (`/bookings/{booking:booking_code}`):**
  - [ ] Live status banner (`wire:poll.10s`):
    - `Confirmed — Ready to Play` (Emerald)
    - `Waitlisted — Position #N in Queue` (Amber, updates automatically if promoted by the Court Owner!)
    - `Checked In` (Blue)
    - `Cancelled / No-Show` (Slate/Red)
  - [ ] Scannable SVG QR Code encoding the verification URL / `booking_code`, venue name, court name, date, time slots, and player name (print/screenshot friendly)
  - [ ] Self-service **"Cancel My Booking / Leave Queue"** button (with confirmation prompt) while status is `confirmed` or `waitlisted`

### Tests & Verification
- [ ] Feature test: Full guest player flow — select available slots -> enter name & phone -> instant confirmation -> Digital QR Booking Pass displays `Confirmed`
- [ ] Feature test: Second player clicks a booked slot -> joins waitlist queue (`Position #1`) -> primary player cancels (or owner marks no-show) -> Court Owner taps "Promote Next in Queue" -> second player's Digital Booking Pass transitions to `Confirmed`
- [ ] Feature test: Two players attempting to book the same available slot concurrently — first succeeds as `Confirmed`, second receives a clean prompt to join the waitlist queue instead

---

## Milestone 2.1 – 2.2: Phase 2 — "Pamisa"-Style Open Play & Paddle Stack Queuing Engine

- [ ] **`OpenPlayQueueService` (`app/Services/OpenPlayQueueService.php`):**
  - [ ] Create capped Open Play sessions (`max_players`, e.g. 16 players, no payment required)
  - [ ] Register player: inside `DB::transaction()`, lock the `OpenPlaySession` row, count active registrations, assign sequential `queue_number`, and set `status = registered` if `< max_players` or `status = waitlist` if full
  - [ ] Auto-promote first `waitlist` player into `registered` when a `registered` player cancels
- [ ] **Live Digital Paddle Stack (`/open-play/{session}` & `/owner/open-play/{session}`):**
  - [ ] Public live board (`wire:poll.5s`) showing **Now Playing on Court (4 players)**, **On Deck / Next Up (4 players)**, and **Paddle Queue Order**
  - [ ] Court Owner / Queue Master controls to check players in and rotate the next 4 players onto the court in one tap (moving finished players to the back of the paddle stack)

---

## Milestone 3.1 – 3.2: Phase 3 — Tournament Bridge & Production Hardening

- [ ] **Tournament Block Booking & Standings Bridge:**
  - [ ] Allow organizers or admins to reserve multi-court blocks under `booking_type = tournament`
  - [ ] Attach or auto-generate the live tournament URL (`tournament_standings_url`) linking to the **Tournament Live Standings** application on the confirmed booking pass and venue schedule
- [ ] **Security Audit & Production Readiness:**
  - [ ] Run security audit across all Livewire actions (role checks, venue ownership checks, daily anti-hoarding rate limits, phone input validation)
  - [ ] Verify N+1 query prevention (`with(['courts', 'bookings.slots'])`) and database indexes
  - [ ] Run full test suite (`php artisan test`), code formatter (`vendor/bin/pint`), and production build (`npm run build`)
