# PINAAK Mobile App — Business Logic & RBAC Specification
### Phase 1 scope: Customer · Vendor (BA) · Driver — UC internal roles excluded

---

## 1. Core Entity & ID Chain

Every transaction in PINAAK flows through one linked ID chain. This is the backbone every role-permission rule hangs off of.

```
ENQUIRY ID (city-wise, auto) ──▶ LEAD ──▶ QUOTATION (RQ)
        │                                    │ (one enquiry can carry
        │                                    │  multiple RQs — different
        │                                    │  vehicles/prices)
        ▼                                    ▼
   BOOKING ID  (= Enquiry ID, once a quote is accepted & paid)
        │
        ▼
   TRIP ID  (Booking ID + suffix a, b, c… — one suffix per vehicle,
             for multi-vehicle bookings e.g. events)
```

**Rule:** A Customer, Vendor, or Driver never sees an ID that isn't linked to them. Visibility is always scoped by this chain, not by a flat "see everything of this type" permission.

---

## 2. Roles & Sub-Roles in This Phase

| Role | Sub-roles / logins | Notes |
|---|---|---|
| **Customer** | Personal (1 login) · Corporate (3 logins: Booking Person, Admin, A/C) · Event (3 logins + Event Vehicle Scheduler) | Corporate/Event logins share one Customer ID but differ in what they can *do* |
| **Vendor (BA)** | Owner · Booking Manager · Ops Manager · A/C Manager | Up to 4 logins per vendor company, same Vendor ID |
| **Driver** | Single role | Attached to one Vendor; assigned to a specific Vehicle per Trip, not permanently |

---

## 3. Business Logic Flow — Stage by Stage

| Stage | What happens (system logic) | Customer | Vendor (BA) | Driver |
|---|---|---|---|---|
| **Enquiry / Lead** | Created on the UC side (call/WA/web/ads) — out of scope this phase | Receives quote notification once UC has qualified the lead | Not involved yet | Not involved |
| **Quotation** | UC creates one or more RQs against the enquiry (different vehicle/price options) | Views quote(s) in a cart-style screen, selects a variant, can request revision (budget remark) or confirm | Not involved | Not involved |
| **Booking** | Quote confirmed + payment (or advance) received → Booking ID generated | Gets booking confirmation, must **acknowledge** it before any driver/vehicle detail is released | Receives **broadcast** (if UC is sourcing vehicle) or direct **assignment** (if BA already tied to route); can Accept / Reject (with reason) / propose vehicle change | Not involved |
| **Vehicle Assignment** | BA accepts → BA assigns vehicle + driver | Sees "vehicle & driver being arranged" status only | Selects vehicle from own fleet, assigns driver, can request UC approval for a vehicle swap | Gets a **new-trip notification** once assigned |
| **Pre-Trip** | Duty Slip generated (BA + Driver), Vehicle Slip generated (to Customer) | Adds passenger list + pickup address (mandatory ~3 days ahead); sees driver & vehicle detail **only after UC approval** | Views/confirms duty slip; runs pre-dispatch checklist | Views duty slip (customer address/phone revealed only close to trip time — privacy rule, see §5) |
| **Trip (live)** | Trip ID active | Views live driver/vehicle location; receives OTP to hand to driver at pickup | Views trip status (upcoming/ongoing) for own fleet only | Views customer's live location; enters the OTP given by customer; captures start/end KM with photo |
| **Payment** | Booking amount %, advance, balance tracked automatically | Views total/paid/balance; pays via gateway; system blocks paying more than the booking amount | Views/confirms BA advance payment; raises GST bill after trip | Confirms customer payment collected (if COD-style component exists) |
| **Feedback** | Mandatory feedback with photo/video after trip closes | Submits feedback (vehicle + driver) | Views own feedback score/history | Not involved |

---

## 4. RBAC Matrices

Legend: **V** = View · **C** = Create · **E** = Edit · **A** = Approve/Confirm · **—** = No access

### 4.1 Customer sub-roles

| Module / Screen | Personal | Corp – Booking Person | Corp – Admin | Corp – A/C | Event – Booking Person | Event – Admin/Scheduler |
|---|---|---|---|---|---|---|
| View Quote | V | V, C(request revision) | V | V | V | V |
| Confirm Booking | A | A | V | — | A | A |
| Event Vehicle Scheduler (multi-vehicle single screen) | — | — | — | — | — | V, C, E |
| Passenger List / Pickup Address | C, E | C, E | V | — | C, E | C, E |
| Live Trip Tracking | V | V | V | — | V | V |
| Payment (view + pay) | V, C | V, C | V | V | V, C | V |
| Feedback | C | C | V | — | C | V |
| Multi-login visibility | n/a | Sees own actions + Admin's overrides | Sees all 3 logins' activity | Sees payment trail only | Sees own actions | Sees all bookings across the event |

### 4.2 Vendor (BA) sub-roles

| Module / Screen | Owner | Booking Manager | Ops Manager | A/C Manager |
|---|---|---|---|---|
| Vendor Profile / Enrolment | V, E | V | V | V |
| Vehicle Master (add/edit vehicle) | C, E, A | V | C, E | V |
| Broadcast — view & respond with rate | V, C | V, C | V | — |
| Booking Accept / Reject | A | A | V | — |
| Assign Driver + Vehicle to Trip | A | V | C, E, A | — |
| Pre-Dispatch Checklist | A | V | C, E, A | — |
| Payments (advance received, weekly status) | V | V | V | V, C |
| Raise GST Bill | A | — | — | C, A |

### 4.3 Driver

Single role — no sub-roles — but two **data-visibility rules** apply (see §5):

| Module / Screen | Access |
|---|---|
| My Trips (Upcoming / Completed / Cancelled) | V — own trips only |
| Duty Slip detail | V — customer contact fields time-gated |
| Customer Live Location | V — only once trip window opens |
| Enter OTP | C — one attempt flow, tied to Trip ID |
| Start/End KM + Photo capture | C — timestamp auto-captured, not editable |
| Driver Briefing checklist | V, A (acknowledge) |

---

## 5. Business Rules That Must Be Enforced in the RBAC Layer (not just UI hiding)

1. **Time-gated visibility, not role-based hiding:** Driver only sees the customer's phone/address once the Duty Slip is issued and trip window is near — this is a *state* condition layered on top of role, not a permanent permission.
2. **Approval gates release information:** Customer never sees Driver/Vehicle detail until UC has approved the assignment — the Customer app must poll/subscribe to an "approved" flag, not just "assigned."
3. **Payment ceiling:** the app must hard-block (not just warn) a Customer from paying more than the current Booking Amount — this is a server-side rule the mobile client should mirror for UX but never rely on client-side alone.
4. **Multi-login = shared entity ID, split permissions:** Corporate/Event Customer logins and Vendor's 4 logins all resolve to the *same* Customer ID / Vendor ID in the data model. RBAC must be enforced per **login (user), not per entity** — two people at the same company can have very different screens.
5. **Vendor can accept a booking without a vehicle number yet** (and add it later) — the Ops Manager permission to "Edit vehicle assignment post-acceptance" must remain open even after the Booking Manager has already approved.
6. **Driver is never permanently tied to a Vendor's specific vehicle** — assignment is per-Trip, so "my vehicle" on the Driver home screen must be trip-scoped, not a fixed profile field.

---

## 6. Screens Used in the Interactive Prototype

| Role | Screens |
|---|---|
| Customer | Home (status) → Quote → Bookings → Trip (live) → Payments → Profile |
| Vendor | Home (broadcasts + new assignments) → Booking Detail (accept/reject/assign) → Vehicles → Payments → Profile |
| Driver | Trips → Duty Slip / Trip Detail (OTP + KM capture) → Profile |

---

## 7. Open Assumptions (flag if wrong)

- Personal Customers are assumed to always have app access in this phase (BRD mentions an "offline system for non-tech-savvy customers" as a separate later requirement, not in scope here).
- Corporate/Event login-splitting is assumed to be enforced at the account/token level (each of the 3 logins has its own credentials, same parent Customer ID).
- Vendor's 4 logins are assumed to work the same way — separate credentials, one Vendor ID.
- COD/cash-collection-by-driver is not explicit in the BRD Payment section; included as a light touch on the Driver payment-confirmation row — confirm if this is real or should be removed.