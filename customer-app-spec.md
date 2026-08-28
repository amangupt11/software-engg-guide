# PINAAK — Customer App: Complete Screens, Flow & Business Logic
### Part 1 of 3 (Vendor and Driver to follow separately)

---

## 0. Gaps Found in the BRD & What I've Added

Cross-checking the module matrix, the Masters matrix, and the 3 real Rate Quotation documents against each other surfaced things the BRD implies but never spells out for the Customer side. Flagged **[GAP]** below; everything else is a direct reflection of what you gave me.

| # | Gap found | Why it matters | What I've added |
|---|---|---|---|
| 1 | BRD says Customer can "View Quote \| Put Remarks \| Confirm" but never describes what happens *after* a remark is put | Without this, a customer's revision request has nowhere to go | A **Quotation negotiation loop** — remarks trigger a new quotation version; customer sees version history |
| 2 | Real quotes show **4–5 vehicle tiers in one document** (Economy/Premium/Royal/Royal VIP), but the BRD's "View Quote" doesn't describe a multi-option compare view | A single "view the quote" screen doesn't work for 5 options with different price/spec | A **tiered comparison screen**, not a single-card view |
| 3 | BRD lists "Request Booking Modify/Cancellation" and "Request Date or Vehicle Change" as customer actions, but never says what happens to that request | A request with no visible status leaves the customer guessing | A **Modification Request tracker** with its own status (Pending / Approved / Rejected) |
| 4 | Real itineraries are **multi-stop, multi-day** (e.g. Mumbai → Trimbakeshwar → Nashik → Shirdi → Aurangabad → Mumbai over 7 days) but the BRD's "Add pickup address" reads like a single address | A multi-city trip has a different pickup point per leg | **Per-leg pickup address**, not a single trip-level address |
| 5 | Corporate quote has a **GST Extra 18%** remark, but no module anywhere mentions a GST invoice being issued to the customer | A Corporate/Event customer needs a downloadable invoice for their accounts team, not just a payment receipt | Added a **GST Invoice download** for Corporate/Event bookings |
| 6 | "Alerts/Notifications" is a listed module but was never designed as a screen | Reminders need somewhere to live and a source of truth | A dedicated **Notification Centre** screen |
| 7 | Nowhere in the BRD is there a **cancellation charge / refund policy** | Customers *will* cancel; if the app doesn't know the rule, this becomes a support-ticket generator | Flagged as an **open business decision** (see §5) — not invented |
| 8 | No **support/help contact** path exists for the Customer app despite TS being referenced elsewhere in your BRD | A stuck customer has no in-app way to reach a human | Added a lightweight **Support screen** (call TS / raise query) |
| 9 | Your Masters matrix shows Customer profile is **Auto-Approved** — but Corporate/Event have 3 logins sharing one profile; nothing says who can edit shared profile fields | Two Corporate logins editing the same address/GST field could conflict | Business rule added: only **Admin login** can edit shared account-level fields; Booking Person/A-C are booking-scoped only |

---

## 1. Customer Journey Map (expanded)

```
Enquiry Exists (view-only, created via UC channels: Call/WA/Email/GAC/GAQ)
        │
        ▼
Quotation Received  ──remarks──▶  UC Revises  ──┐
        │  ▲__________________________________ ┘
        │  (loop until customer accepts or quote expires)
        ▼
Vehicle Tier Selected & Confirmed
        │
        ▼
Advance Payment Paid  ──▶  Booking Created (Booking ID)
        │
        ▼
Booking Acknowledged  (driver/vehicle stay hidden until this happens)
        │
        ├──▶ Modification Request (date/vehicle/cancel) ──▶ Pending ──▶ Approved/Rejected
        │
        ▼
Passenger List + Per-Leg Pickup Address added
        │
        ▼
Pre-Trip: Driver & Vehicle Revealed (once UC approves assignment)
        │
        ▼
Trip Live: Location Tracking, OTP Handover
        │
        ▼
Balance Payment Paid  ──▶  (Corporate/Event: GST Invoice available)
        │
        ▼
Feedback Submitted
```

---

## 2. Screens — Purpose, Data, Actions, Business Rules

### 2.1 Login / Account Type Selection
- **Personal**: single credential set.
- **Corporate / Event**: 3 logins share one account — **Booking Person**, **Admin**, **A/C**. Login screen doesn't ask "which role" — the role is bound to the credential itself.
- **Business rule:** Admin is the only login that can edit account-level fields (billing address, GSTIN, company name). Booking Person can create/manage individual trip bookings but not touch account settings. A/C is read-only except for the Payments screen.

### 2.2 Home / Dashboard
- **Shows:** active Enquiry status (if no quote yet), newest Quotation needing action, nearest upcoming Trip, any pending reminder (payment due, passenger list due).
- **Business rule:** Home always surfaces exactly one "next action" card — never more than one — to avoid decision paralysis when a Corporate account has 5 bookings in flight at once.

### 2.3 Enquiry Status *(new — read-only)*
- **Shows:** Enquiry ID, date raised, channel it came through (Call/WA/Email/Ads), current stage ("Being quoted by our team").
- **No create action here** — per your instruction, Enquiry/Lead capture lives in the UC Admin Portal. This screen exists only because your BRD lists "View Enquiry" as a customer capability; it's a status mirror, not an input form.
- **Open question for you:** should this screen exist at all in v1, given the app's real entry point is Quotation? I've included it as optional/toggleable — easy to cut.

### 2.4 Quotation — Tiered Comparison *(expanded)*
- **Shows:** all vehicle tiers from the single Quotation document side-by-side — seat count, vehicle model, tier name (Economy/Premium/Royal/Royal VIP), tier description, discounted price, price validity date/countdown.
- **Actions:** Select a tier → Confirm, or **Put Remarks** (free-text: "budget is tighter", "need AC upgrade") without picking a tier yet.
- **Business rule (negotiation loop):** Putting a remark does **not** close the quotation — it creates a new Quotation *version* once UC/TS responds. Customer sees a small version history ("v1 sent → your remark → v2 sent"). Only the latest version is actionable; older versions are read-only history.
- **Business rule (expiry):** If price validity date passes with no confirmation, the quote auto-expires and the customer sees "Quote expired — request a fresh quote" (routes to Support, not a new Enquiry — Enquiry creation is out of app scope).
- **GST note:** for Corporate accounts, each tier's price is shown with a "+18% GST" tag beside it, not baked into the headline number — so a Corporate viewer isn't surprised at payment time.

### 2.5 Bookings List
- **Shows:** Upcoming and Past, each with Booking ID, route summary, dates, status chip.
- **Business rule:** Booking ID always equals the Quotation/Enquiry ID — this is a hard identity rule your backend must enforce, not just a display convenience, since Vendor/Driver apps reference bookings by this same ID.

### 2.6 Booking Detail
- **Shows:** full itinerary (multi-leg), acknowledgement status, vehicle tier confirmed, payment summary.
- **Action — Acknowledge:** required before driver/vehicle details unlock anywhere downstream (Trip screen, notifications).
- **Action — Request Modification** *(expanded from BRD's bare mention)*: three sub-types —
  - **Cancel Booking** — customer states a reason; goes to Pending status.
  - **Change Date** — customer proposes a new date; goes to Pending status.
  - **Change Vehicle Tier** — customer picks a different tier from the same quotation; goes to Pending status.
- **Business rule:** all three modification types go through the **same status machine** (Pending → Vendor/UC Approved or Rejected), so the customer always sees one consistent tracker instead of three different UIs. Rejected requests show the reason UC/Vendor gave.
- **Business rule (cancellation charge):** **not defined in your BRD.** I have *not* invented a percentage or slab here — the UI should show "Cancellation charges may apply as per policy" as a placeholder until you confirm the actual slab (e.g. free before X days, 50% within Y days, etc.).

### 2.7 Passenger List & Per-Leg Pickup Address *(expanded from single-address assumption)*
- **Shows:** one row per itinerary leg (e.g. "Leg 1: Mumbai pickup — 2 Aug, 6:00 AM", "Leg 3: Nashik pickup — 4 Aug"), each with its own address field, plus one shared passenger list for the whole trip.
- **Business rule:** the system should nudge this ~3 days before the **first** leg's date, not the whole-trip start, since some multi-day itineraries have the customer joining partway through (event/corporate group travel).

### 2.8 Trip — Live
- **Shows:** driver name & photo, vehicle number & photo, live location, OTP (large, tap-to-copy).
- **Locked state:** shows "Details available once your booking is acknowledged and approved" — this is the same gate as 2.6, surfaced here too so the customer isn't confused mid-trip-prep.
- **Business rule:** OTP is generated per Trip ID (not per Booking ID) — a multi-vehicle booking (event with 3 buses) has 3 separate OTPs, one per vehicle/trip leg.

### 2.9 Payments
- **Shows:** Total, Paid, Balance; payment history log (not just a single "paid" flag — multiple partial payments are normal for advance + balance + any mid-trip top-up).
- **Action:** Pay Balance — hard-capped at the balance amount (never allow overpayment).
- **Corporate/Event addition:** **GST Invoice** download button, enabled once the trip is marked complete and full payment is received.
- **Business rule:** invoice numbering/format is a finance-system concern, not something this app should generate itself — the app should call an invoicing service/endpoint rather than construct GST documents inline.

### 2.10 Feedback
- **Shows:** simple rating for driver + vehicle + overall trip, free-text, optional photo/video attachment.
- **Business rule:** feedback submission unlocks after trip status = Completed, not before.

### 2.11 Notification Centre *(new — was a listed module with no screen)*
- **Shows, per your BRD's Alerts module:** "Add trip details" reminder, "Pending payment" reminder — plus, from the gaps above: "New quotation version available," "Modification request update," "Feedback pending."
- **Business rule:** notifications are generated by backend events (quote revised, request approved/rejected, payment due date) — never client-side timers, so they survive app reinstalls/device changes.

### 2.12 Profile & Account
- **Shows:** account type (Personal/Corporate/Event), for Corporate/Event: which of the 3 logins is currently active and what it can do.
- **Business rule:** see 2.1 — Admin-only fields are visibly locked/greyed for Booking Person and A/C logins, not just permission-blocked silently.

### 2.13 Support *(new)*
- **Shows:** "Call your TS" (tap-to-call, using the TS assigned to this customer's city/enquiry), "Raise a query" (free text, tagged to Booking ID if opened from a booking context).
- **Business rule:** this is a routing screen, not a resolution engine — it hands off to a human (TS), it doesn't try to be a chatbot.

---

## 3. State Machines

**Quotation:** `Sent → (Remarks → Revised → Sent)* → Confirmed | Expired`

**Booking:** `Pending Ack → Acknowledged → [Modification Requested → Approved | Rejected] → Assigned → Ongoing → Completed | Cancelled`

**Payment:** `Unpaid → Partially Paid → Fully Paid` (Refund path not modeled — see open items)

---

## 4. Open Items Needing a Business Decision (not invented, flagged for you)

1. **Cancellation & refund policy** — no slab/percentage exists anywhere in the BRD. The app needs one before "Cancel Booking" can be more than a request-and-wait button.
2. **Whether the Enquiry Status screen (2.3) ships in v1** — it's read-only and low-risk, but if the real entry point is Quotation, it may be simpler to cut it entirely for the first release.
3. **GST invoice generation** — confirm whether this is a separate finance/accounting system your backend calls, or something PINAAK itself needs to generate and number.
4. **Quotation version limit** — is there a cap on how many remark-rounds a quote can go through before it's escalated to a human call instead of another revision?

---

*Next: Vendor app screens, flow & business logic — say the word and I'll do that one next, same format.*