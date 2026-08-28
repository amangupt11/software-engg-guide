# PINAAK — Driver App: Complete Screens, Flow & Business Logic

### Part 3 of 3 (Customer and Vendor done — this completes the set)

---

## 0. Gaps Found in the BRD & What I've Added

| # | Gap found                                                                                                                                                                                                                      | Why it matters                                                                                                                                                  | What I've added                                                                                                           |
| - | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| 1 | Masters matrix shows a driver can be**added directly by the Vendor** (pre-approved) *or* **self-register** via their own login (needing Vendor/UC approval) — but no screen distinguishes these two entry paths | A driver opening the app for the first time needs to know which situation they're in                                                                            | Login/Registration screen (§2.1) covers both paths explicitly                                                            |
| 2 | Trip module only mentions "Enter Start & End KM" once per trip — but your real itineraries are multi-day, multi-stop (e.g. a 7-day round trip through 6 cities)                                                               | One start-KM and one end-KM for a week-long trip loses the day-by-day record that both the vendor and any fuel/maintenance accounting would need                | Expanded to**per-day (per-leg) KM capture**, matching the itinerary chain already established on the Customer side  |
| 3 | Nothing anywhere describes what happens if a driver**can't do an assigned trip** (illness, emergency, breakdown)                                                                                                         | Vendor assigns a driver, but the BRD gives the driver no way to flag "I can't take this"                                                                        | Added a**Trip Decline / Report Unavailability** action, distinct from the Vendor's own driver-change action         |
| 4 | Both Vendor ("Customer Payment - View/Confirm during trip") and Driver ("Customer Payment - View/Confirm Details") appear to confirm the*same* payment event                                                                 | Unclear whether this is one confirmation two people can see, or two independent confirmations that could disagree                                               | Flagged as an**open item** (§5) — needs your call on who is the actual collector vs. who just sees a mirror       |
| 5 | No mid-trip**driver handoff** process exists anywhere, despite Vendor being able to "Change Driver" on an active multi-day trip                                                                                          | On a 7-day trip, if the driver changes on day 4, the outgoing driver's app needs to stop showing the trip and the incoming driver's needs to pick it up cleanly | Added a**handoff transition** to the Trip state machine (§3)                                                       |
| 6 | Driver documents are limited to "Self Photo\| Driving License" with no mention of license expiry tracking                                                                                                                      | A driver whose license expires mid-assignment is a real safety/compliance risk, and nothing currently watches for it                                            | Added a**license expiry reminder** to the Notification Centre — suggested addition, not a stated requirement       |
| 7 | No safety/emergency feature (SOS) exists anywhere across any document you've shared, despite drivers running long intercity routes                                                                                             | This is a common expectation in transport apps and worth a decision either way                                                                                  | Flagged as a**suggested addition** (§5) — not built in, since it wasn't asked for, but worth your explicit yes/no |

---

## 1. Driver Journey Map

```
Added by Vendor (auto-approved)  OR  Self-registers in app (Pending Vendor/UC Approval)
        │
        ▼
Approved & Active
        │
        ▼
Trip Assigned by Vendor (vendor decides — driver has no "pick" action)
        │
        ├─▶ Decline / Report Unavailable ──▶ Vendor re-assigns a different driver
        │
        ▼ (Accept — implicit by proceeding)
Pre-Trip Briefing / Checklist Acknowledged
        │
        ▼
Duty Slip Viewed — customer contact details locked until 1 day before trip date
        │
        ▼
Per-Leg Trip Execution:
   Day 1: Start KM (photo) → drive → End KM (photo)
   Day 2: Start KM (photo) → drive → End KM (photo)
   ... (repeats per itinerary leg)
        │
        ▼
OTP entered at each pickup point (per Trip ID / vehicle)
        │
        ▼
Customer Payment sighted/confirmed during trip (see open item on dual-confirmation)
        │
        ▼
Trip marked Completed
```

---

## 2. Screens — Purpose, Data, Actions, Business Rules

### 2.1 Login / Registration *(new — both entry paths covered)*

- **Path A — Added by Vendor:** driver receives login credentials already created by their Vendor (Owner/Ops Manager); account is active immediately, no approval wait.
- **Path B — Self-registration:** driver signs up directly, uploads self photo + driving license, and sits in **Pending Approval** until either the Vendor or UC approves — whichever acts first.
- **Business rule:** a driver in Pending status can log in and see their own profile/status, but cannot be assigned to any trip.

### 2.2 Home / Dashboard

- **Shows:** next assigned trip (if any), any pending reminder (license expiring, briefing not yet acknowledged).
- **Business rule:** same "one next action" principle as the Customer Home screen — a driver mid-multi-day-trip shouldn't be shown unrelated future trips as if they need action now.

### 2.3 My Trips

- **Shows:** Upcoming / Ongoing / Completed / Cancelled, each trip tagged with its Trip ID (Booking ID + leg suffix).
- **Business rule:** a trip only appears here once the Vendor has assigned this specific driver to it — there is no "available trips" list for drivers to browse (consistent with the corrected Vendor-side model: nothing is picked, everything is assigned).

### 2.4 Trip Detail / Duty Slip

- **Shows:** full multi-leg itinerary, pickup points per leg, vehicle assigned.
- **Customer contact (name, phone):** locked, shows "Unlocks 1 day before trip date" until that gate passes — matching the Masters matrix rule exactly.
- **Live location:** driver's own location is shared with the customer once the trip window opens (mirrors the Customer app's live-tracking screen).
- **Action — Enter OTP:** entered once per Trip ID at the pickup point, given verbally by the customer.
- **Action — Decline / Report Unavailable** *(new, per §0.3):* flags this assignment back to the Vendor with a reason (illness, breakdown, personal emergency) — this pulls the driver off the trip and notifies the Vendor to reassign, distinct from the Vendor proactively choosing to change drivers.

### 2.5 Start/End KM Capture *(expanded to per-leg, per §0.2)*

- **Shows:** one Start KM + End KM entry **per day/leg** of the itinerary, not one pair for the whole multi-day trip.
- **Action:** photo of odometer + auto-captured timestamp, for both start and end of each leg.
- **Business rule:** entries are append-only once submitted — a driver can't retroactively edit a KM photo/timestamp after the fact, since this is the record both the vendor and any fuel/billing reconciliation rely on.

### 2.6 Payments

- **Shows:** customer payment collected during the trip — view/confirm.
- **[GAP — open item]** whether the Driver's confirmation here is the primary record (driver physically collects cash/UPI) or a secondary mirror of what the Vendor already confirmed is unresolved — see §5.

### 2.7 Pre-Trip Briefing / Checklist

- **Shows:** statutory warnings, prohibited items, vehicle condition acknowledgement.
- **Business rule:** this was in the original prototype based on general good practice, not something explicitly named in the two BRD matrix images you shared this round — flagging it as **carried over**, not newly invented from these specific documents, in case you want to verify it's still wanted.

### 2.8 Profile

- **Shows:** own details, license number/expiry, attached Vendor (view-only — per Masters matrix, driver can view but not edit Vendor Master), current vehicle (trip-scoped, not a fixed profile field).
- **Business rule:** "my vehicle" is never a permanent field on this screen — it only ever reflects whatever vehicle the current/most recent trip assigned, since drivers aren't fixed to one vehicle.

### 2.9 Notification Centre

- **Shows, per BRD:** "Reminder — Upcoming Trip: Date & Time."
- **Added (§0.6):** license expiry reminder (suggested, not stated in BRD), duty slip issued notification, driver-handoff notification (if this driver is being swapped out or newly assigned mid-trip).

### 2.10 Support

- Same pattern as Customer/Vendor — routes to a human (Vendor or UC contact), not a resolution engine.

---

## 3. State Machines

**Driver Approval:** `Added by Vendor (auto-approved) | Self-registered (Pending Vendor/UC Approval) → Approved → Active`

**Trip Assignment:** `Assigned by Vendor → Acknowledged | Declined → (Declined: Vendor reassigns) → Briefing Acknowledged → Per-Leg Execution (Start KM → Drive → End KM)× N legs → Completed`

**Mid-Trip Driver Handoff** *(new, per §0.5):* `Vendor initiates Change Driver → Outgoing driver's app removes trip access → Incoming driver's app gains trip access from the current leg onward → Customer is notified of the new driver/vehicle detail`

---

## 4. Open Items Needing a Business Decision

1. **Payment dual-confirmation** — is the Driver the actual collector (cash/UPI in hand) with the Vendor just viewing a mirror, or can either confirm independently? This affects whether a mismatch between the two is even possible, and if so, how it's resolved.
2. **Per-leg KM granularity** — confirmed here as one Start/End KM pair per day/leg of a multi-day trip; please confirm this matches how vendors actually want to reconcile fuel/mileage, versus a single trip-level pair.
3. **SOS / emergency feature** — not built in, since it wasn't part of any document you've shared, but worth an explicit yes/no given the intercity, multi-day nature of these trips.
4. **License expiry enforcement** — should an expiring/expired license actually **block** new trip assignments for that driver, or just generate a reminder? The former is a harder guardrail; the latter just nudges.
5. **Driver decline reason categories** — should "illness / breakdown / personal emergency" be a fixed dropdown (cleaner for reporting) or free text?

---

*That completes all three roles — Customer, Vendor, Driver. Want me to now pull these three specs together into one consolidated cross-role document, or move on to something else (e.g. the actual DB schema / API endpoint design from here)?*
