# AGENTS.md

Guidance for AI coding agents working on the **CourtPass / RacketHub** repository. Read this fully before making changes.

---

## Project Overview

**CourtPass / RacketHub** is a pure multi-venue court reservation, per-slot waitlist queuing, open-play paddle queuing, and tournament venue booking platform for **Pickleball** and **Badminton**, inspired by the slot-allocation and crowd-queuing workflow of our church (*"Pamisa"*) reservation system — **with zero payment processing, zero fee/markup math, and zero financial ledgers**.

* **Anyone with the public link (Players / Organizers)** can browse partner venues, view the live hourly court availability matrix, book available slots with **instant confirmation** (up to the venue's daily per-player hour cap), join a **per-slot Waitlist Queue** if a time slot is already booked, receive a Digital Booking Pass with a verification QR code and live queue status, self-cancel if plans change, or register for capped Open Play drop-in sessions (no player login required).
* **Court Owners (`court_owner`)** log into a fast, mobile-first companion portal scoped strictly to their venue to view their daily court agenda, tap **"Instant 1-Tap Block"** for physical walk-in players, scan player booking QR codes at check-in, mark cancellations or no-shows, and tap **"Promote Next in Queue"** to manually promote waitlisted players when a slot opens up.
* **Super Admins (`super_admin` — Management)** onboard partner venues, configure courts and operating hours, set daily per-player booking caps, monitor global bookings and waitlist queues across all venues, inspect audit logs, and bridge Tournament Block bookings to the **Tournament Live Standings** app.

### Scope by Roadmap Phase

* **Phase 1 (MVP Pilot — Core Court Booking & Slot Waitlist Queue):** 1–2 pilot venues, dynamic hourly slot booking matrix, instant confirmation for available slots, per-slot waitlist queue with manual Court Owner promotion, Digital QR Booking Pass with self-cancel, Court Owner 1-Tap Walk-in Blocker, and Super Admin venue/court setup.
* **Phase 2 (Community Open Play & Paddle Stack Queuing Engine):** *"Pamisa"*-style capped Open Play drop-in sessions (e.g., max 16 players), automated session waitlists, and live digital paddle stack / 4-player on-court rotation board.
* **Phase 3 (Expansion & Tournament Bridge):** Multi-court Tournament Block bookings with automated linking to the Tournament Live Standings app, plus multi-city venue filters and production hardening.

**Out of scope (do not build unless explicitly asked):** payment gateways (PayMongo/Xendit/GCash), manual payment receipt uploads, pricing/markup/discount calculators, payout settlement ledgers, 30-day reschedule credit tokens, native iOS/Android mobile apps, player social networks/chat, equipment e-commerce store, or modifying tournament bracket rules inside this repository.

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

Roles live in a `role` column on `users` (`court_owner`, `super_admin`), plus a nullable `venue_id` foreign key on `users` that binds a `court_owner` to their specific venue. Players book or join queues as public guests using their name, phone number, and an unguessable `booking_code` pass URL.

| Capability | Public Player | Court Owner (`court_owner`) | Super Admin (`super_admin`) |
| :--- | :---: | :---: | :---: |
| Browse venues, courts & live slot matrix | Yes | Yes | Yes |
| Instantly reserve available hourly slots & view QR pass | Yes | Yes | Yes |
| Join per-slot Waitlist Queue on booked slots & self-cancel | Yes | Yes | Yes |
| Register for Open Play sessions & view live paddle queue | Yes | Yes | Yes |
| View daily court agenda & scan booking QR for check-in | No | Own Venue Only | All Venues |
| **1-Tap Quick Block** / unblock slots for walk-in players | No | Own Venue Only | All Venues |
| Mark booking as Cancelled / No-Show & **Promote Next in Queue** | No | Own Venue Only | All Venues |
| Adjust venue operating hours & daily per-player slot cap | No | Own Venue Only | All Venues |
| Onboard venues, manage courts & assign court owners | No | No | Yes |
| Attach Tournament Live Standings bridge URLs | No | Own Venue Only | All Venues |
| View global Audit Logs & master bookings/queue monitor | No | No | Yes |

**Authorization & Multi-Tenant Isolation Rules:**
1. Authorization is enforced server-side inside every Livewire action (Gate/Policy or explicit role + venue check), never only via route middleware or hidden buttons.
2. Every `court_owner` query and action **must** verify `$venue->id === auth()->user()->venue_id`. A Court Owner must never read or mutate another venue's courts, bookings, or queues.

---

## Data Model (8 Core Tables — Zero Financial Tables)

1. **`venues`**: `id`, `name`, `slug` (unique), `address`, `lat` (nullable), `lng` (nullable), `contact_phone`, `rules` (nullable), `operating_open_time` (default `06:00`), `operating_close_time` (default `22:00`), `max_daily_hours_per_player` (unsigned tiny int, default `2`), `status` (`active` | `inactive`).
2. **`courts`**: `id`, `venue_id`, `name` (e.g. `Court 1`), `sport` (`pickleball` | `badminton`), `surface` (e.g. `taraflex`, `wood`, `acrylic`, `concrete`), `is_active` (boolean).
3. **`bookings`**: `id`, `booking_code` (unique, e.g. `CP-20261124-X8F2`), `venue_id`, `user_name`, `phone`, `booking_type` (`hourly` | `open_play` | `tournament` | `walk_in`), `status` (`confirmed` | `waitlisted` | `checked_in` | `cancelled` | `no_show`), `queue_position` (unsigned small int, nullable — set when `status = waitlisted`), `tournament_standings_url` (nullable), `cancelled_reason` (nullable).
4. **`booking_slots`**: `id`, `booking_id`, `court_id`, `date` (date), `start_time` (time), `end_time` (time), `active_lock_key` (nullable string/virtual key set only when parent booking is `confirmed` or `checked_in`, enforcing a unique database constraint on active `(court_id, date, start_time)` while allowing multiple `waitlisted` rows for the same slot).
   * **Concurrency Invariant:** At most one active primary booking (`confirmed` or `checked_in`) can hold a `(court_id, date, start_time)` coordinate at any time.
5. **`open_play_sessions`**: `id`, `venue_id`, `court_id`, `session_date`, `start_time`, `end_time`, `max_players` (int, e.g. `16`), `status` (`open` | `full` | `in_progress` | `completed` | `cancelled`).
6. **`open_play_registrations`**: `id`, `session_id`, `booking_id` (nullable), `player_name`, `contact`, `queue_number` (int), `status` (`registered` | `waitlist` | `checked_in` | `on_court` | `cancelled`).
7. **`audit_logs`**: `id`, `user_id` (nullable), `venue_id` (nullable), `action`, `subject`, `old_values` (JSON nullable), `new_values` (JSON nullable), `created_at`.
8. **`users`**: standard Laravel table plus `role` (`court_owner` | `super_admin`) and `venue_id` (nullable FK to `venues`).

---

## Business Rules (Do Not Change Without Asking)

1. **Zero Double-Booking Invariant:**
   * All slot reservations, queue promotions, and walk-in blocks must run inside `DB::transaction()` using row locks (`lockForUpdate()`) and a database-level uniqueness constraint (`active_lock_key`) so two concurrent requests can never confirm the same `(court_id, date, start_time)`.
2. **Instant Confirmation & Daily Anti-Hoarding Limit:**
   * Because there is no payment checkout step, selecting an available slot and submitting a name + phone immediately creates a `confirmed` booking.
   * To prevent a single player from hoarding free slots all day, `SlotBookingService` enforces `$venue->max_daily_hours_per_player` (default `2` hours per phone number per venue per date across active `confirmed`, `checked_in`, and `waitlisted` slots).
3. **Per-Slot Waitlist Queue & Manual Court Owner Promotion:**
   * When a slot is already `confirmed` or `checked_in`, another player can join that slot's **Waitlist Queue**.
   * Each waitlisted booking receives the next sequential `queue_position` (`1, 2, 3...`) for that `(court_id, date, start_time)` with `status = waitlisted` and `active_lock_key = null`.
   * When a `confirmed` booking is cancelled (by the player via their Digital Booking Pass or by the Court Owner marking `cancelled` / `no_show`), the slot's active lock is released.
   * **Manual Promotion by Court Owner:** The Court Owner sees the queue indicator on their Daily Court Agenda and taps **"Promote Next in Queue"**, which atomically transitions the next waitlisted booking into `status = confirmed` and locks the slot.
4. **Court Owner "Instant 1-Tap Block" for Walk-ins:**
   * Caretakers on-site can block any open slot on their venue's daily agenda in one tap.
   * A walk-in block creates a `bookings` row with `booking_type = walk_in` and `status = confirmed`, locking the slot in `booking_slots` immediately. Caretakers can also unblock a `walk_in` slot if tapped by mistake.
5. **Open Play Capped Sessions & Paddle Stack Queue ("Pamisa" Style):**
   * Each Open Play session has a strict `max_players` cap (e.g. 16 players).
   * Registrations up to `max_players` get `status = registered`. Registrations after the cap is reached receive `status = waitlist` ordered by `queue_number`.
   * If a registered player cancels, the first `waitlist` player is promoted into `registered`.
   * During the session, the Live Digital Paddle Stack tracks **Now Playing on Court (4 players)**, **On Deck / Next Up (4 players)**, and the remaining queue order.
6. **Tournament Standings Bridge:**
   * When a `tournament` block booking is confirmed, the system can store or generate a deep link (`tournament_standings_url`) to the **Tournament Live Standings** application.
7. **Audit Trail:**
   * Every walk-in block/unblock, booking cancellation/no-show, manual queue promotion, check-in, and venue/court setting change is recorded in `audit_logs`.

---

## Architecture

```text
app/
  Models/            Venue, Court, Booking, BookingSlot, OpenPlaySession,
                     OpenPlayRegistration, AuditLog, User
  Enums/             Sport, UserRole, BookingType, BookingStatus,
                     VenueStatus, OpenPlayStatus, RegistrationStatus
  Services/          SlotAvailabilityService, SlotBookingService,
                     WalkInBlockerService, OpenPlayQueueService, AuditLogger
  Livewire/          Public/, Owner/, Admin/
  Http/Middleware/   EnsureUserHasRole, EnsureOwnerHasVenueAccess
```

* Keep all business logic in `app/Services`, never inside Livewire components or Blade views.
* `SlotAvailabilityService`, `SlotBookingService`, `WalkInBlockerService`, and `OpenPlayQueueService` are core domain services and must be thoroughly unit tested (including race conditions, daily slot caps, waitlist queue ordering, manual queue promotion, and walk-in blocks).

---

## Frontend and UX (RALLY Brand & Color Theme)

> **Design Source of Truth:** Always follow [docs/rally_color_theme_rationale.md](./docs/rally_color_theme_rationale.md) for the official **RALLY** color palette, contrast rules, and component treatments.

* **RALLY Color Palette (Tailwind Theme Tokens):**
  * **Deep Forest Green (`#173F32` / `rally-forest`):** Primary brand foundation (~60% visual anchor) — navigation bar, footer, dark hero headers, major headings on light backgrounds, and dark text on lime buttons.
  * **Court Green (`#24764E` / `rally-court`):** Secondary interactive & availability green — `"Available"` slot status indicators, active nav states, secondary buttons, and icons.
  * **Electric Lime (`#C8F169` / `rally-lime`):** High-energy accent & CTA (~10% share) — primary `"Book a Court"` / `"Confirm Court Booking"` buttons, currently **selected** date/time slots, and key action badges. **Always pair with dark text (`#173F32` or `#252B28`) for WCAG contrast; never use white text on lime.**
  * **Warm Ivory (`#F4F1E7` / `rally-ivory`):** Main page canvas (~30% share) — comfortable warm off-white page and section background behind cards.
  * **Soft White (`#FFFFFF` / `rally-surface`):** Card surfaces — court cards, booking forms, slot grid containers, queue drawers, modals, and dashboard panels.
  * **Charcoal (`#252B28` / `rally-charcoal`):** Main body text, form labels, and supporting details on light surfaces.
* **Status Semantics & Accessibility:**
  * Never communicate slot status by color alone; always pair colors with explicit text labels or icons (`"Available"`, `"Selected"`, `"Booked · Join Queue (#N)"`, `"Walk-In Blocked"`).
  * Every interactive slot button must have a minimum `44x44px` touch target and visible hover, focus, selected, and disabled states.
* **Portal 1 (Player Matrix):** Warm Ivory (`#F4F1E7`) page canvas with Deep Forest Green (`#173F32`) header and Soft White (`#FFFFFF`) court cards. Visual hourly grid per court (`Court Green #24764E` = `"Available"`, `Electric Lime #C8F169` with `#173F32` text = `"Selected"`, `Amber` = `"Booked · Tap to Join Waitlist Queue"` with queue count badge, `Muted Gray` = `"Unavailable / Walk-In Blocked"`). Sticky bottom bar with Electric Lime CTA for quick name + phone submission.
* **Portal 2 (Court Owner Companion):** Zero clutter. Defaults directly to **Today's Court Grid** with 1-tap `Block Walk-In` on empty slots, 1-tap `Promote Next in Queue` (highlighted with Electric Lime `#C8F169` CTA when waitlisted players are waiting on an open slot), and 1-tap `Check In` / `Mark No-Show` on booked slots.
* **Live Updates:** Use `wire:poll.8s` on the public slot matrix and owner daily agenda so newly booked, waitlisted, promoted, or walk-in-blocked slots update automatically without page refresh.

---

## Working Agreement

1. Follow the build order in `DEV_PLAN.md`: migrations & enums -> models & seeders -> core slot-booking, waitlist & walk-in services with unit tests -> auth & tenant middleware -> Super Admin portal -> Court Owner companion -> Public Player booking & waitlist flow -> Open Play & Paddle Stack Queue -> Tournament Bridge.
2. If a request conflicts with the Business Rules above, stop and ask instead of working around it.
3. Never commit `.env` or hardcoded credentials. Seed initial accounts from `.env` variables (`SUPER_ADMIN_INITIAL_PASSWORD`, `PILOT_OWNER_INITIAL_PASSWORD`).
4. Run `php artisan test` and `vendor/bin/pint` after every implementation phase.
