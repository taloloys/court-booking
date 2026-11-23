# CourtPass / RacketHub System Plan
### *Multi-Venue Court Reservation, Open Play Queuing & Tournament Platform for Pickleball and Badminton*

---

| Metadata | Details |
| :--- | :--- |
| **Document Classification** | `PROJECT PROPOSAL & SYSTEM ARCHITECTURE` |
| **Concept Inspiration** | Church *"Pamisa"* Model |
| **Primary Sports** | Pickleball & Badminton |
| **Business Model** | Markup (Add-on) / Discount (Margin) |
| **Target Stack** | Laravel + Livewire + Tailwind CSS |

---

> [!IMPORTANT]
> **Management Directive:**  
> *"Uso man pud pickle ball kron.. think of something nga pareho sa simbahan.. saboton nako ang mga owners ug court.. then pa book for use or tournament mo add on ta or ila iless atoa.... tnx kaayo"*

---

## 1. Executive Summary & Problem-Solution Matrix

The racquet sports market in the Philippines is undergoing unprecedented growth. While Badminton has an entrenched, multi-decade player base, **Pickleball is experiencing viral community adoption** across public and private clubs. However, court booking operations remain primitive, fragmented, and manual.

| Domain | Current Market Pain Point | Platform Solution (Our System) |
| :--- | :--- | :--- |
| **Court Booking** | Messy FB Messenger group chats, manual GCash receipt verification, constant double-booking with walk-in players. | **Live Self-Service Slot Calendar:** Real-time available hours per court, instant digital reservation, automated slot holds. |
| **Open Play & Queuing** | Manual paper rosters or chat lists (*"Listahan: 1. Juan, 2. Pedro"*); chaotic court rotation disputes during drop-in nights. | **"Pamisa"-Style Slot Allocation:** Capped participant registration (e.g. 16 max), live digital paddle stack & rotation board. |
| **Tournaments** | Organizers struggle to block multiple courts across venues and manually run brackets on spreadsheets. | **Tournament Venue Booking + Direct Integration** with our built-in **Live Standings & Fixture Tracker**. |

---

## 2. The "Pamisa" Analogy: How the Church Model Maps to Courts

Our existing church/pamisa system successfully mastered **slot allocation, crowd scheduling, and fee collection**. We adopt the exact same core mechanics for sporting venues:

| Church / Pamisa System | Court Booking Equivalent | Operational Logic & Value |
| :--- | :--- | :--- |
| **Parishes / Chapels** | **Partner Venues / Court Facilities** | Court owners onboard their venue, specify court count (e.g., Court 1–4), sport type, and surface (taraflex/wood). |
| **Mass Times / Schedules** | **Court Time Slots (1-Hour Blocks)** | Operating hours (e.g., 6:00 AM – 10:00 PM) split into bookable hourly slots with peak and off-peak pricing. |
| **Mass Intention Types** | **Booking Service Types** | Three offerings: **(1) Casual Hourly Court**, **(2) Open Play Pass** (per head), or **(3) Tournament Block**. |
| **Mass Stipend / Offering** | **Court Rental Fee + Platform Margin** | Automated payment processing with transparent commission/markup breakdown before receipt issuance. |
| **Parish Admin Roster** | **Court Owner "One-Tap" Portal** | Mobile dashboard for court owners to see today's bookings and block slots instantly for walk-in players. |

---

## 3. Revenue & Monetization Engine ("Add-on or Less")

Management specified two operational partnership agreements with court owners:

> ### Model A: The "Add-On" (Convenience Markup)
> *Best for independent court owners who insist on receiving their standard published rates.*
> 
> * **Court Owner Standard Rate:** `₱300 / hour`
> * **Platform Convenience Fee:** `+ ₱30 / hour`
> * **Player Checkout Price:** `₱330 / hour`
> * **Settlement:** Court receives `₱300`; Platform keeps `₱30` pure profit.

> ### Model B: The "Discount / Margin" ("Ila i-less atoa")
> *Best for partner courts wanting high volume. Players pay standard rate; platform receives wholesale margin.*
> 
> * **Court Regular Published Rate:** `₱300 / hour`
> * **Partner Discounted Rate to Us:** `₱260 / hour`
> * **Player Checkout Price:** `₱300 / hour` *(Zero markup to player)*
> * **Settlement:** Court receives `₱260`; Platform keeps `₱40` profit.

---

## 4. System Architecture & Core Portals

### 📱 Portal 1: Player Booking Web App
*Public, ultra-responsive mobile experience*
* **Venue Discovery:** Filter by location, sport, and amenities
* **Live Slot Matrix:** Real-time view of Available vs. Reserved hours
* **Cart Hold:** 10-minute temporary cart slot hold
* **Instant Checkout:** GCash / Maya / QRPh direct integration
* **Digital Pass:** Mobile booking pass with verification QR code

### 🛠️ Portal 2: Court Owner Companion
*Fast, no-fuss mobile interface for staff*
* **Daily Agenda:** Timeline view of all court schedules
* **"Quick Block" Button:** One-tap button for physical walk-in players
* **QR Scanner:** Camera check-in scanner for arriving players
* **Earnings & Payouts:** Daily summary & payout disbursement ledger
* **Court Controls:** Operating hours and pricing adjustments

### 📊 Portal 3: Super Admin (Manager View)
*Platform governance and partner accounting*
* **Venue Onboarding:** Partner onboarding & court configuration
* **Agreement Selector:** Toggle between Add-on markup vs wholesale discount
* **Financial Ledger:** Master transaction and financial reports
* **Payout Management:** Automated payout calculation per court
* **Tournament Bridge:** Direct bridge to Tournament Standings tool

---

## 5. Relational Database Schema Architecture

Designed for clean separation of concerns, high concurrency (preventing double-bookings), and auditability:

| Table | Primary Fields & Types | Purpose & Business Rule |
| :--- | :--- | :--- |
| `venues` | `id, name, slug, address, lat, lng, contact_phone, rules, status` | Stores partner court facilities and location coordinates. |
| `courts` | `id, venue_id, name, sport (badminton\|pickleball), surface, is_active` | Specific physical playing courts inside a venue. |
| `court_rates` | `id, court_id, day_of_week, start_time, end_time, hourly_rate, is_peak` | Configures dynamic peak and off-peak pricing per hour. |
| `partner_agreements` | `id, venue_id, model (markup\|discount), markup_val, discount_val, bank_details` | Controls the manager's revenue model per venue. |
| `bookings` | `id, booking_code, user_name, phone, booking_type, court_fee, fee_addon, grand_total, status (pending\|confirmed\|cancelled), hold_expires_at` | Master booking record with temporary 10-min slot hold to prevent race conditions during checkout. |
| `booking_slots` | `id, booking_id, court_id, date, start_time, end_time, price` | Specific 1-hour blocks reserved. Enforces unique index on `(court_id, date, start_time)` to prevent double-booking. |
| `open_play_sessions` | `id, venue_id, court_id, session_date, start_time, end_time, max_players, fee_per_player` | "Pamisa"-style community open play sessions with capped capacity. |
| `open_play_registrations` | `id, session_id, player_name, contact, queue_number, status` | Individual player queue and attendance tracker. |
| `payout_settlements` | `id, venue_id, date_from, date_to, total_court_earnings, status, reference_no` | Disbursement tracking for court owners. |

---

## 6. Integration Bridge: Court Booking + Tournament Standings

How this new system seamlessly links with your existing **Tournament Standings Tracker**:

* **Step 1: Venue Tournament Booking**  
  An organizer selects a partner venue (e.g. *"Metro Sports Club - 4 Courts"*) for Saturday, 8:00 AM – 5:00 PM under a **Tournament Package**.

* **Step 2: Instant Standings Activation**  
  Upon payment confirmation, the system creates the tournament profile and auto-provisions a link to our **Tournament Live Standings App** for live scoring and standings!

---

## 7. Four-Phase Implementation Roadmap

| Phase | Target Scope | Deliverables & Milestone | Timeline |
| :---: | :--- | :--- | :---: |
| **Phase 1**<br>*(MVP Pilot)* | **Single-Venue Proof of Concept** | • Onboard 1-2 friendly pilot court owners.<br>• Basic slot booking matrix (pick date & time).<br>• Manual GCash receipt upload & verification.<br>• Mobile slot blocker for the court owner. | **Weeks 1 – 3**<br>*(Fastest Value)* |
| **Phase 2**<br>*(Automation)* | **Automated Payments** | • Integrate PayMongo / Xendit (Instant GCash/Maya confirmation).<br>• Automated 10-minute slot hold & release timer.<br>• Automated weekly court owner payout ledger. | **Weeks 4 – 6** |
| **Phase 3**<br>*(Community)* | **Open Play & Queuing Engine** | • "Pamisa"-style Open Play drop-in sessions.<br>• Player cap management & automated waitlists.<br>• Live digital paddle stack & on-court rotation display. | **Weeks 7 – 8** |
| **Phase 4**<br>*(Expansion)* | **Tournament Bridge & Multi-City** | • Deep integration with Tournament Standings Tracker.<br>• Multi-city court expansion across Visayas & Mindanao.<br>• Club membership passes & monthly subscription add-ons. | **Month 3+** |

---

## 8. Risk Analysis & Operational Safeguards

### Risk 1: Walk-in Double Booking
* **Risk:** Court caretaker books a walk-in player locally while an online player books the same slot on the website.
* **Safeguard:** The Court Owner Companion app has a prominent **"Instant 1-Tap Block"** button that caretakers can tap in 2 seconds from their phone. In addition, slot holds lock slots for 10 minutes during checkout.

### Risk 2: Partner Court Trust & Prompt Payouts
* **Risk:** Court owners become skeptical about receiving their rental money collected by our platform.
* **Safeguard:** Transparent daily ledger in the partner dashboard. Guaranteed scheduled weekly or bi-weekly bank/GCash disbursements with zero hidden deductions.

### Risk 3: Player No-Shows & Weather Disruptions
* **Risk:** Players reserve slots and don't show up, or outdoor courts get rained out.
* **Safeguard:** Full payment or non-refundable downpayment required at booking. In case of certified rain/power outages, court owners can issue a 1-click **"Reschedule Token"** valid for 30 days.

### Risk 4: Technical Scope Creep
* **Risk:** Overcomplicating Phase 1 by building native mobile apps or complex ERPs prematurely.
* **Safeguard:** Build as a lightweight, lightning-fast Mobile-First Responsive Web App (PWA) using Laravel + Livewire + Tailwind. Zero app store installation barrier for players.

---

## 9. Conclusion

This system plan directly translates your intuition into an actionable, highly profitable sports marketplace. By taking the proven slot-allocation mechanics of the church/pamisa system and applying it to the booming Pickleball and Badminton scene.
