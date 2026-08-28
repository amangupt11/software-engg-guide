# PINAAK — Vendor (BA) App: Complete Screens, Flow & Business Logic
### Part 2 of 3 (Customer done; Driver to follow)

---

## 0. Gaps Found in the BRD & What I've Added

| # | Gap found | Why it matters | What I've added |
|---|---|---|---|
| 1 | BRD's "View Available Bookings (Broadcast by UC) / Pick Bookings" reads like an in-app multi-vendor marketplace | **Corrected by you:** the "broadcast" is a manual process — UC calls around to check vendor availability/rate over the phone, then manually assigns one vendor. Nothing competitive happens inside the app. | Removed the separate Broadcast Pool screen entirely — every booking a vendor sees in-app is already a **Direct Assignment**, decided by a human before it reaches the app |
| 2 | "Vehicle Availability Screen — Block Veh. for Others" is listed as one bullet with no detail on *how* a vendor blocks a vehicle | A vendor needs to take a vehicle off the market for a date range (own use, maintenance) — that's a calendar action, not a toggle | Added a proper **Vehicle Availability Calendar** screen |
| 3 | "Confirm/Reject/Modify Booking/Change Vehicle" — "Modify Booking" scope was never defined (can vendor change price? dates? or only logistics?) | Left undefined, this could accidentally let a vendor edit pricing, which should never be a vendor permission | Scoped explicitly: vendor's "Modify" = **logistics only** (vehicle/driver/pickup timing), never price or itinerary — flagged in §5 to confirm |
| 4 | Two different "Payment" line items exist in your BRD — **"Customer Payment - View/Confirm"** and **"Vendor Payment - View/Confirm"** — with no explanation that these are two separate money flows | Conflating "money the customer paid" with "money the vendor gets paid" is the kind of bug that causes real accounting disputes | Split into two ledgers on the Payments screen: **Customer Payment Collected** (on UC's behalf) vs. **Vendor Payout/Settlement** (what UC owes the vendor) |
| 5 | "Feedback — Customer Feedback / UC Feedback" for Vendor — unclear whether "UC Feedback" means UC rating the vendor, or the vendor rating UC | Wrong assumption here misdirects a whole screen | Assumed: vendor **views** customer ratings about them, and **submits** feedback about UC's service/process — flagged to confirm in §5 |
| 6 | Masters matrix shows Vendor Master is **View only** in the Vendor app — but nothing in the BRD explains how a *new* vendor gets onto the platform at all | Mirrors the Customer-side Enquiry gap — vendor onboarding needs to live somewhere | Assumed vendor onboarding happens entirely in the **UC Admin Portal**, same boundary as Enquiry — flagged to confirm |
| 7 | Vendor has 4 logins (Owner/Booking Mgr/Ops Mgr/A-C Mgr) sharing one account, but no rule for who gets notified about what | Without routing, either everyone gets flooded or the right person misses something | Added a **notification-routing rule** by sub-role (§2.11) |
| 8 | "Assign/Change Driver \| Assign Vehicle" then separately "Request to Change Vehicle \| Approved by UC" — reads like two different approval requirements for the same action | This is actually an important asymmetry: assigning a vehicle the *first* time is free; *changing* it afterward needs UC's sign-off | Called out explicitly as a business rule in §2.8, not left ambiguous |

---

## 1. Vendor Journey Map (expanded)

```
Vendor & Fleet Onboarded (UC Admin Portal — outside this app)
        │
        ▼
Vehicle & Driver added by Vendor  ──▶  Pending UC Approval  ──▶  Approved / Rejected
        │
        ▼
Booking arrives as a Direct Assignment
        │        (UC has already manually chosen and called this vendor —
        │         nothing competitive happens inside the app)
        ▼
   Accept / Reject  ──(Reject)──▶  sent back to UC, who manually calls the next vendor
        │ (Accept)
        ▼
        Assign Vehicle + Driver (free) ──▶ later Change Vehicle (needs UC approval)
                 │
                 ▼
        Duty Slip Issued, Pre-Dispatch Checklist
                 │
                 ▼
        Trip Runs — Vendor monitors status (Upcoming/Ongoing/Completed)
                 │
                 ▼
        Customer Payment Collected (view/confirm) ── separate from ──▶ Vendor Payout Settlement
                 │
                 ▼
        GST Bill Raised (post-trip)
                 │
                 ▼
        Views Customer Feedback  |  Submits Feedback about UC
```

---

## 2. Screens — Purpose, Data, Actions, Business Rules

### 2.1 Login / Sub-Role
- One Vendor ID, up to 4 logins: **Owner** (full access), **Booking Manager** (accept/reject bookings), **Ops Manager** (vehicle/driver assignment), **A/C Manager** (payments/billing only).
- **Business rule:** permission is bound to which login was used, enforced server-side per request — never inferred from a client-side "current role" flag alone.

### 2.2 Home / Dashboard
- **Shows (varies by sub-role):** Owner/Booking Mgr see new assignments needing accept/reject; Ops Mgr sees bookings needing vehicle/driver assignment; A/C Mgr sees pending payouts and unbilled completed trips.
- **Business rule:** each sub-role's home screen surfaces only the actions *that role* can act on — no dead-end cards showing an action the logged-in role can't perform.

### 2.3 Assignments *(corrected — no broadcast/pick screen exists)*
- **Shows:** bookings UC has manually decided and assigned to this vendor — every booking a vendor ever sees in-app arrives this way, whether UC's internal process called it a "direct assignment" or a "broadcast check" on their end.
- **Actions:** Accept / Reject (with reason). That's the full extent of the vendor's decision — no bidding, no rate negotiation, no competing with other vendors inside the app.
- **Business rule:** Reject sends the booking back to UC, who manually calls the next vendor — the app has no visibility into or role in that re-assignment process; it simply waits for the next assignment to appear, if this vendor is chosen again or a different one is.

### 2.4 Vehicle Master
- **Shows:** vendor's fleet list, each vehicle's approval status (Pending UC Approval / Approved / Rejected).
- **Actions:** Add/Edit vehicle, upload vehicle images.
- **Business rule (from Masters matrix):** every new vehicle and every vehicle image needs **UC approval** before the vehicle becomes bookable — a vehicle in "Pending" status cannot be offered to UC or assigned to a booking.

### 2.5 Vehicle Availability Calendar *(new — expanded from a one-line bullet)*
- **Shows:** a calendar per vehicle with day-by-day status: Available / Blocked (vendor-set) / On Trip / Reserved (mid-negotiation hold).
- **Action:** Block a date range (own use, maintenance) — this tells UC (during their manual availability check) that the vehicle can't be offered for those dates.
- **Business rule:** a vehicle already assigned to a confirmed trip cannot be blocked for those same dates — the system should reject that action rather than silently create a conflict.

### 2.6 Driver Master
- **Shows:** vendor's driver list, license details, approval status.
- **Actions:** Add/Edit/Approve a driver directly (per Masters matrix, vendor *can* self-approve a driver — no mandatory UC gate here, unlike Vehicle).
- **Business rule:** if the driver instead self-registers via the Driver app, that registration needs approval from **either** the Vendor or UC (whichever acts first) — not both required.

### 2.7 Trip Monitoring & Duty Slip
- **Shows:** Upcoming / Ongoing / Completed / Cancelled trips for this vendor's fleet, Duty Slip detail per trip.
- **Actions:** Assign Driver + Vehicle (first time — no approval needed), Change Driver (no approval needed), **Change Vehicle after initial assignment (requires UC approval)**, run pre-dispatch checklist.
- **Business rule (the asymmetry called out in §0.8):** initial vehicle assignment is a vendor-only decision; *changing* the vehicle once it's been assigned and communicated to the customer needs UC sign-off, since the customer may already have seen the original vehicle's details.

### 2.8 Payments — Two Ledgers *(split, per §0.4)*
- **Ledger A — Customer Payment Collected:** view/confirm payment the customer has made during the trip (cash/UPI collected on UC's behalf) — this is money passing *through* the vendor, not money owed *to* the vendor.
- **Ledger B — Vendor Payout/Settlement:** view booking advance received from UC, raise GST bill after trip completion, track settlement status.
- **Business rule:** these two ledgers must never be shown as one merged number — a vendor confirming "customer paid ₹5,000" is a completely different fact from "UC owes me ₹18,000 for this trip."
- **[GAP — open item]** whether vendor payout happens per-trip or as a periodic (e.g. weekly) settlement isn't defined anywhere — flagged in §5.

### 2.9 Feedback
- **Shows:** ratings/comments customers have left about this vendor's vehicles and drivers (view-only).
- **Action:** submit feedback about UC's process (assignment turnaround, support responsiveness) — **assumption, flagged in §5** since "UC Feedback" wasn't defined either direction in your BRD.

### 2.10 Notification Centre *(new — routing rule added)*
- **Routing rule by sub-role:**
  - New assignment alerts → **Owner + Booking Manager**
  - Driver/vehicle assignment reminders → **Owner + Ops Manager**
  - Payment/payout alerts → **Owner + A/C Manager**
- **Business rule:** Owner receives everything; the other three sub-roles receive only what's relevant to their function, so a Booking Manager isn't pinged about a payout issue they have no permission to act on.

### 2.11 Company Profile (view only)
- **Shows:** vendor company details, registration documents, logo — **view only**, per the Masters matrix; edits happen in the UC Admin Portal.
- **[GAP — open item]** since there's no in-app edit path, how does a vendor request a correction to their own profile? Flagged in §5.

### 2.12 Support
- Same pattern as the Customer app's Support screen — routes to a human (UC Ops contact), not a resolution engine.

---

## 3. State Machines

**Booking (vendor side):** `Assigned by UC (manual) → Accept | Reject → (Reject: UC manually calls next vendor) → Vehicle+Driver Assigned → [Change Vehicle → UC Approval] → Duty Slip → Ongoing → Completed | Cancelled`

**Vehicle:** `Added → Pending UC Approval → Approved | Rejected → Available | Blocked | On Trip`

**Driver:** `Added by Vendor (auto-approved) | Self-registered (Pending Vendor/UC Approval) → Approved → Active`

**Vendor Payout:** `Booking Accepted → Advance Received → Trip Completed → GST Bill Raised → Settled`

---

## 4. Notification Routing Summary

| Event | Owner | Booking Mgr | Ops Mgr | A/C Mgr |
|---|---|---|---|---|
| New assignment | ✓ | ✓ | — | — |
| Driver/vehicle assignment due | ✓ | — | ✓ | — |
| Payment/payout update | ✓ | — | — | ✓ |
| Vehicle approval status change | ✓ | — | ✓ | — |

---

## 5. Open Items Needing a Business Decision

1. **"Modify Booking" scope** — confirmed here as logistics-only (vehicle/driver/pickup timing), never price or itinerary. Please confirm this is correct before it's built as a permission boundary.
2. **Vendor payout cadence** — per-trip settlement vs. periodic (e.g. weekly) batch settlement — not defined anywhere in the BRD.
3. **"UC Feedback" direction** — confirmed assumption is vendor-submits-feedback-about-UC; please confirm or correct.
4. **Vendor profile correction path** — since Vendor Master is view-only in-app, how does a vendor request a change to their own company details?
5. **Commission/rate model** — since UC negotiates rate with the vendor manually (by phone) before assignment, does the app need to display/store that agreed rate anywhere on the Assignment screen, or does it stay entirely a UC-side/Admin-Portal concern?

---

*Next: Driver app screens, flow & business logic — say the word.*