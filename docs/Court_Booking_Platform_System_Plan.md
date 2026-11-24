# CourtPass / RacketHub System Plan
### *Pure Multi-Venue Court Reservation, Per-Slot Waitlist Queue, Open Play Paddle Stack & Tournament Platform for Pickleball and Badminton*

---

| Metadata | Details |
| :--- | :--- |
| **Document Classification** | `PROJECT PROPOSAL & SYSTEM ARCHITECTURE (PURE BOOKING & QUEUE EDITION)` |
| **Concept Inspiration** | Church *"Pamisa"* Slot Allocation & Queuing Model |
| **Primary Sports** | Pickleball & Badminton |
| **Operational Model** | Instant Court Reservation + Per-Slot Waitlist Queue + Open Play Paddle Queue (Zero Payment Processing) |
| **Target Stack** | Laravel 13 + Livewire + Tailwind CSS |

---

## 1. Executive Summary & Problem-Solution Matrix

The racquet sports market is experiencing rapid growth across Badminton and **Pickleball**. However, court reservation and on-court queuing remain chaotic, relying on messy group chats and paper lists. This platform focuses **100% on friction-free court booking and queuing** without payment friction or billing complexity.

| Domain | Current Market Pain Point | Platform Solution (Our System) |
| :--- | :--- | :--- |
| **Court Booking & Slot Waitlist** | Messy FB Messenger group chats, constant double-booking with walk-in players, and no structured waitlist when someone cancels or no-shows. | **Live Self-Service Slot Calendar + Per-Slot Waitlist Queue:** Real-time available hours per court, instant booking confirmation, daily per-player slot cap, and a per-slot waitlist queue that Court Owners can promote in 1 tap. |
| **Open Play & Paddle Queuing** | Manual paper rosters or chat lists (*"Listahan: 1. Juan, 2. Pedro"*); chaotic court rotation disputes during drop-in nights. | **"Pamisa"-Style Slot Allocation:** Capped participant registration (e.g. 16 max), automated waitlist, and live digital 4-player paddle stack & rotation board. |
| **Tournaments** | Organizers struggle to block multiple courts across venues and connect schedules to live tournament brackets. | **Tournament Venue Block Booking + Direct Integration** with our **Tournament Live Standings & Fixture Tracker**. |

---

## 2. The "Pamisa" Analogy: How the Church Model Maps to Courts

We adopt the slot-allocation and crowd-scheduling mechanics of our church/pamisa system directly for sporting venues:

| Church / Pamisa System | Court Booking Equivalent | Operational Logic & Value |
| :--- | :--- | :--- |
| **Parishes / Chapels** | **Partner Venues / Court Facilities** | Court owners onboard their venue, specify court count (e.g., Court 1–4), sport type, surface (taraflex/wood), and daily per-player hour limit. |
| **Mass Times / Schedules** | **Court Time Slots (1-Hour Blocks)** | Operating hours (e.g., 6:00 AM – 10:00 PM) split into bookable hourly slots with instant confirmation and a per-slot waitlist queue. |
| **Mass Intention Types** | **Booking Service Types** | Three offerings: **(1) Hourly Court Reservation**, **(2) Open Play Queue Pass**, or **(3) Tournament Block**. |
| **Parish Admin Roster** | **Court Owner "One-Tap" Portal** | Mobile dashboard for court owners to see today's bookings, block slots in 1 tap for walk-in players, and manually promote waitlisted players when a slot opens. |

---

## 3. Core Queuing & Anti-Hoarding Mechanics (Zero-Payment Architecture)

Because booking is free of online payment friction, the platform uses three clean operational rules to keep court utilization fair and conflict-free:

1. **Instant Confirmation + Daily Anti-Hoarding Limit:**
   * Selecting an available hourly slot and entering a player's Name + Mobile Phone confirms the slot immediately (`status = confirmed`) and issues a **Digital QR Booking Pass**.
   * Each venue defines `max_daily_hours_per_player` (default `2` hours/day per phone number) so no single player can hoard an entire court all day.
2. **Per-Slot Waitlist Queue & Manual Owner Promotion:**
   * If a slot is already booked, additional players can tap **"Join Waitlist Queue"** for that specific `(court, date, time)` slot and receive a live queue rank (`Position #1`, `Position #2`, etc.).
   * If the primary player cancels via their Digital Pass—or if the Court Owner marks them as **Cancelled / No-Show** on-site—the Court Owner taps **"Promote Next in Queue"** to immediately confirm the next waitlisted player.
3. **Open Play Capped Queue & Live 4-Player Paddle Stack:**
   * Open Play drop-in sessions cap active registrations (e.g. `max_players = 16`) and place overflow players on an ordered waitlist.
   * On court night, the **Live Digital Paddle Stack** displays **Now Playing (4 players)**, **On Deck / Next Up (4 players)**, and the rotation queue on any phone or TV screen.

---

## 4. System Architecture & Core Portals

### 📱 Portal 1: Player Booking & Queue Web App
*Public, ultra-responsive mobile experience*
* **Venue Discovery:** Filter by location, sport (`Pickleball` / `Badminton`), and surface
* **Live Slot Matrix:** Real-time view of `Available` (Instant Book) vs. `Booked` (Tap to Join Waitlist Queue) vs. `Walk-In Blocked`
* **Digital QR Pass:** Mobile booking pass with scannable QR code, live waitlist queue position (`wire:poll`), and self-service **Cancel Booking / Leave Queue**
* **Open Play Board:** Join capped Open Play sessions and watch the live 4-player paddle rotation board

### 🛠️ Portal 2: Court Owner Companion
*Fast, no-fuss mobile interface for staff*
* **Daily Agenda:** Timeline view of all court schedules (`wire:poll.8s`)
* **"1-Tap Quick Block" Button:** One-tap button to lock/unblock open slots for physical walk-in players
* **Waitlist Queue Promoter:** View queued players on any slot, mark primary bookings as `No-Show / Cancelled`, and tap **"Promote Next in Queue"**
* **QR Check-In Scanner:** Camera or code check-in for arriving players
* **Open Play Queue Master:** 1-tap rotation controls to cycle the next 4 players onto the court

### 📊 Portal 3: Super Admin (Manager View)
*Platform governance and multi-venue oversight*
* **Venue & Court Onboarding:** Configure venues, courts, surfaces, operating hours, and daily per-player slot limits
* **Global Booking & Queue Monitor:** Live overview of bookings and waitlist queues across all partner venues
* **Audit Trail:** Full audit log of walk-in blocks, queue promotions, cancellations, and configuration updates
* **Tournament Bridge:** Direct bridge linking Tournament Block bookings to the **Tournament Live Standings** app

---

## 5. Relational Database Schema Architecture (8 Lean Tables)

Designed for high concurrency (preventing double-bookings), clear queue ordering, and auditability:

| Table | Primary Fields & Types | Purpose & Business Rule |
| :--- | :--- | :--- |
| `venues` | `id, name, slug, address, lat, lng, contact_phone, rules, operating_open_time, operating_close_time, max_daily_hours_per_player, status` | Stores partner court facilities, operating window, and daily per-player hour cap. |
| `courts` | `id, venue_id, name, sport (badminton\|pickleball), surface, is_active` | Specific physical playing courts inside a venue. |
| `bookings` | `id, booking_code, venue_id, user_name, phone, booking_type, status (confirmed\|waitlisted\|checked_in\|cancelled\|no_show), queue_position, tournament_standings_url` | Master booking or slot-waitlist record. |
| `booking_slots` | `id, booking_id, court_id, date, start_time, end_time, active_lock_key` | Specific 1-hour blocks. `active_lock_key` enforces a unique index on active `(court_id, date, start_time)` while allowing multiple waitlisted rows per slot. |
| `open_play_sessions` | `id, venue_id, court_id, session_date, start_time, end_time, max_players, status` | "Pamisa"-style community open play sessions with capped capacity. |
| `open_play_registrations` | `id, session_id, booking_id, player_name, contact, queue_number, status` | Individual player queue, waitlist, and on-court rotation tracker. |
| `audit_logs` | `id, user_id, venue_id, action, subject, old_values, new_values, created_at` | Immutable operational audit log. |
| `users` | `id, name, email, password, role (court_owner\|super_admin), venue_id` | Staff accounts scoped by role and venue. |

---

## 6. Three-Phase Implementation Roadmap

| Phase | Target Scope | Deliverables & Milestone | Timeline |
| :---: | :--- | :--- | :---: |
| **Phase 1**<br>*(MVP Pilot)* | **Core Court Booking & Slot Waitlist Queue** | • Onboard 2 pilot venues with Pickleball & Badminton courts.<br>• Live slot booking matrix with instant confirmation & daily anti-hoarding cap.<br>• Per-slot Waitlist Queue + Court Owner 1-tap **"Promote Next in Queue"**.<br>• Mobile **"1-Tap Walk-In Blocker"** & Digital QR Booking Pass with self-cancel. | **Weeks 1 – 3**<br>*(Fastest Value)* |
| **Phase 2**<br>*(Community)* | **Open Play & Paddle Stack Queuing Engine** | • "Pamisa"-style capped Open Play drop-in sessions (e.g. 16 players max).<br>• Automated session waitlist promotion on cancellation.<br>• Live Digital Paddle Stack & 4-player on-court rotation board. | **Weeks 4 – 5** |
| **Phase 3**<br>*(Expansion)* | **Tournament Bridge & Multi-City** | • Multi-court Tournament Block booking linked to **Tournament Live Standings**.<br>• Multi-city venue filters, security audit, and Laravel Cloud deployment. | **Week 6+** |
