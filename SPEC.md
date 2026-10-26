# Renewal Radar — Product Spec v1

**One-line pitch:** *Recover two lapsed members a month and this app pays for itself.*

---

## 1. The problem (say it in the owner's words)

Gym owners track memberships in one of three ways:

1. A **paper register** — they must manually scan it to find who's expiring
2. **Excel/Google Sheets** — accurate but almost never opened
3. **Nothing** — *"my members come back on their own"*

The consequence is **silent churn**. A member stops coming in month 4, expires quietly, and nobody calls. The owner only notices when the monthly total drops — by which time the member has joined the gym down the road.

**The math that makes this sellable:**

| Gym size | Monthly revenue | One lost member = |
|---|---|---|
| 100 members @ ₹1,200 | ₹1.2 lakh | ₹14,400/year |
| 200 members @ ₹1,500 | ₹3 lakh | ₹18,000/year |
| 300 members @ ₹1,000 | ₹3 lakh | ₹12,000/year |

Losing 5 members a year to silent churn costs a 200-member gym **₹90,000**. The app costs ₹12,000/year.

---

## 2. The wedge (and the non-goals)

### In scope for v1
- Member list with expiry date
- Automated WhatsApp reminder 7 days before expiry, on expiry day, and 3 days after
- One screen showing: lapsed / expiring / renewed / PT-low
- Rupee values on every number
- One-tap "Mark renewed" (extends expiry, resets PT)
- PT session ledger (total + remaining)

### Explicitly OUT of scope — refuse these
| Feature | Why not |
|---|---|
| Attendance / biometrics | Hardware + support nightmare |
| Diet plans, workout plans | Content business, not software |
| Payroll, staff management | Different buyer, different app |
| Payment collection / UPI | RBI + reconciliation + support load |
| Member-facing mobile app | The member gets WhatsApp. That IS the app. |
| Accounting / GST | Tally's job |
| Multi-branch | v2 at the earliest, needs a different pitch |

**Every feature added dilutes the pitch.** The whole product is one screen.

---

## 3. Who uses it

| | Gym owner (buyer + user) | Member (never logs in) |
|---|---|---|
| Age | 28–50 | Any |
| Tech comfort | Low. Uses WhatsApp heavily, Instagram for marketing | — |
| Device | Android phone at the desk | Any |
| What they need | To know, in 5 seconds, who to chase today | A WhatsApp message they can reply to |

**Design constraint: the owner must understand the screen without training.** If it needs an explanation, it's wrong. Big numbers, plain words, one action per row.

---

## 4. Screens

### 4.1 Dashboard (the only screen that matters)

```
┌─────────────────────────────────────────┐
│  💪 Renewal Radar                       │
│     Iron House Gym · Andheri            │
├─────────────────────────────────────────┤
│  4          3          7          2     │
│  Lapsed     Expiring   Renewed    PT    │
│  ₹4,800     ₹3,900     ₹8,400     low   │
│  lost       at risk    recovered        │
├─────────────────────────────────────────┤
│ [Needs action 9] [Lapsed 4] [Soon 3]... │
├─────────────────────────────────────────┤
│  Rahul Sharma                           │
│  +91 98765 43210 · Monthly · ₹1,200     │
│  [Expired 4d ago]  [PT 2/12]            │
│  [💬 WhatsApp]  [✓ Mark renewed]         │
│  Last reminded 6d ago                   │
├─────────────────────────────────────────┤
│  Priya Nair ...                         │
└─────────────────────────────────────────┘
```

**Rules:**
- Stats row always visible — it's the emotional hook
- Default tab is **"Needs action"** — the owner never has to hunt
- Every row has exactly 2 buttons
- Rows colour-coded: red = lapsed, amber = expiring, green = active

### 4.2 Add member
6 fields: name, WhatsApp number, plan, start date, PT total, PT left.
Expiry is **computed**, never typed. Owner only knows the start date.

### 4.3 Edit member
Same six fields, but the date field becomes **Expiry date** and is edited directly.
Editing a lapsed member forward resets the reminder history.

### 4.4 Bulk import (v1, essential)
Must accept a paste of a member list (CSV or "Name, Phone, Plan" lines).
**Owner will not type 150 members.** You type them in for the first customer if needed.

Accepted per line: `Name, Phone, Plan, Start date` — last two optional.
Plan names normalise (`Half-yearly` → `halfyearly`). Invalid lines are skipped and counted out, not silently dropped.

### 4.5 Soft delete vs hard delete
Two distinct actions, and the distinction is a feature:

| Action | Effect | When the owner uses it |
|---|---|---|
| **Mark as left** | `status = 'left'`. Hidden from all lists and from money-at-risk. Reversible. | Member moved cities, quit, injury |
| **Delete** | Row removed permanently. | Data-entry mistake, duplicate, test row |

Without this split, every departed member sits in "Lapsed" forever and the headline rupee number becomes noise the owner stops trusting. Reactivate restores a left member with their original expiry.

### 4.6 Search
Single fuzzy match on **name or phone number**, applied on top of the active tab.
Debounced 150ms. Clearing restores the tab.

### 4.7 Sort
A single button beside the search box that cycles **Urgency → A–Z → Z–A**.

| Mode | Order | When the owner wants it |
|---|---|---|
| **Urgency** (default) | Soonest expiry / most overdue first | Daily triage — this is the working mode |
| **A–Z** | Alphabetical by name | Looking someone up, reading down the register |
| **Z–A** | Reverse alphabetical | Rare, but free once A–Z exists |

Members marked *left* stay pinned to the bottom in every mode — otherwise a departed member lands in the middle of the list and the owner re-reads it every time. Sort choice persists in `localStorage`.

### 4.8 Flag
A per-member boolean marking "keep an eye on this person".

- Shown as a **gold star** before the name and an **★ Unflag** toggle in the row actions
- Gives a dedicated **Flagged** tab
- Also surfaces the member under **Needs action** even if their membership is otherwise healthy

This last point is the whole reason the feature exists. The tabs answer *"what's expiring?"*; the flag answers *"who do I need to ring?"* — the member who's stopped showing up, the one asking about cancelling, the lead still deciding. Those people have valid expiry dates and would never appear in an expiry-driven list.

Keep it a manual judgement call. **Do not** auto-flag from heuristics (missed payments, low session counts) until there is real attendance data — a wrong auto-flag teaches the owner to ignore the star.

### 4.9 Note
A single free-text note per member — the context that expires nowhere and lives in nobody's head after a week.

- Edited via **✎ Add note** / **✎ Note** in the row actions, or by tapping the note itself
- Renders on the card as a tinted block, clamped to **3 lines** (full text in the editor)
- Included in **search** alongside name and phone
- Emptying the note removes the block entirely rather than leaving a blank box

What owners actually write:

| Note | Why it matters |
|---|---|
| "Asked about pausing in Nov — travelling" | Turns a silent churn into a scheduled phone call |
| "Referred by Hannah — don't lose him" | Identifies members whose loss costs more than one membership |
| "Wants PT package at renewal — quote 10-session rate" | The reason the renewal call happens at all |
| "Left Oct — moved to Leeds, maybe back after Christmas" | Keeps history on *left* members, who are the warmest leads you have |

**One field, not many.** A structured "reason for leaving" dropdown sounds better and gets used less — owners won't fill forms, but they will type a sentence. Do not add timestamps, tags, or note history in v1; the moment it needs explaining, it stops being used.

**Search must cover notes.** This is the sleepers feature — the real query is *"who was the one who wanted to pause?"*, not *"what was their name?"* Without note search, the note is only visible to someone who already remembers who it belongs to.

### 4.10 Gym details
Gym name and location are **settings, not constants**. Tap the name in the header to edit.

The name is used in two places and must stay in sync:

1. The header subtitle (`Name · Location`)
2. **Every WhatsApp message** — `"your {gymName} membership expires in 3 days"`

That second one is why this matters. A demo with someone else's gym name in the message reads as a template; the owner's own name reads as their tool. **Set the real gym name before you hand over the phone.**

Location is optional and cosmetic (separator disappears when blank). The name is required — saving blank is rejected rather than silently leaving an empty header.

### 4.11 Settings
Gym name (appears in WhatsApp messages), reminder timing offsets, message templates.

---

## 5. Data model

```sql
CREATE TABLE gyms (
  id            uuid PRIMARY KEY,
  name          text NOT NULL,
  phone         text,
  owner_phone   text NOT NULL,        -- login via OTP
  reminder_days int[]  DEFAULT '{7,0,-3}',
  created_at    timestamptz DEFAULT now()
);

CREATE TABLE members (
  id            uuid PRIMARY KEY,
  gym_id        uuid REFERENCES gyms ON DELETE CASCADE,
  name          text NOT NULL,
  phone         text NOT NULL,        -- 10 digits, no country code
  plan          text NOT NULL,        -- monthly|quarterly|halfyearly|yearly
  start_date    date NOT NULL,
  expiry_date   date NOT NULL,        -- computed on insert, indexed
  pt_total      int DEFAULT 0,
  pt_left       int DEFAULT 0,
  status        text DEFAULT 'active',-- active|paused|left
  last_reminded date,
  notes         text,
  created_at    timestamptz DEFAULT now()
);

CREATE INDEX ON members (gym_id, expiry_date);

CREATE TABLE renewals (
  id         uuid PRIMARY KEY,
  member_id  uuid REFERENCES members ON DELETE CASCADE,
  amount     int NOT NULL,
  renewed_at timestamptz DEFAULT now()
);

CREATE TABLE messages (
  id         uuid PRIMARY KEY,
  member_id  uuid REFERENCES members ON DELETE CASCADE,
  kind       text,      -- upcoming|today|overdue|manual
  body       text,
  sent_at    timestamptz DEFAULT now(),
  delivered  boolean,
  replied    boolean
);
```

**Design notes:**
- `expiry_date` is stored, not computed at read time — makes the daily cron a simple indexed query
- PT sessions live on the member, not a separate table, in v1. A session ledger is v2.
- No PHI concerns beyond name + phone. Keep it that way.

---

## 6. WhatsApp — pick one of three

This is the make-or-break integration. Three options at very different price points:

| Option | Cost | Automation | Setup |
|---|---|---|---|
| **A. `wa.me` deep links** (what the prototype does) | **Free** | ❌ Manual tap per member | Zero. Works today. |
| **B. WhatsApp Cloud API** (Meta direct) | ~₹0.10–0.35 per template message in India | ✅ Fully automated | Business verification, template approval (2–7 days) |
| **C. BSP like Interakt / Wati / AiSensy** | ₹2,000–4,000/mo + per-message | ✅ Automated + UI | Easiest, but eats your margin at ₹999 pricing |

### Recommendation: **Ship A, then upgrade to B.**

- **v1 with A.** Owner taps "WhatsApp", message is pre-filled, they hit send. Manual, but *completely free* and live on day one. Most gyms with <100 members will accept this.
- **v2 with B** once you have 10 paying customers. Automate. This is the feature that justifies ₹1,999/month.

**Important:** WhatsApp Business **template messages** must be pre-approved by Meta. Get approval started on **day 1** — it takes up to a week and will block your launch otherwise.

Template to submit:

```
Hi {{1}}, your {{2}} membership expires on {{3}}. 
Renew to keep your streak going. Reply here to renew.
```

---

## 7. Pricing

| Plan | Price | Includes |
|---|---|---|
| **Starter** | **₹999/month** | Up to 200 members, manual WhatsApp, dashboard |
| **Growth** | **₹1,999/month** | Unlimited members + automated WhatsApp + PT ledger |
| **Yearly** | 10 × monthly | 2 months free — push this, it fixes your cash flow |

**Why ₹999:**
- Below the ₹1,000 psychological line
- 3× cheaper than the desktop-era gym software owners already resent
- One saved renewal (₹1,200–2,500) is more than a month

**Do not offer a free tier.** It attracts the wrong gyms and kills your support time. Offer **first 30 days free, no card**, then charge. Different thing.

**Discount lever for the first 10:** ₹499/month locked for life in exchange for a testimonial + 2 referrals. You are buying proof, not revenue.

---

## 8. Tech stack (smallest thing that works)

| Layer | Choice | Why |
|---|---|---|
| Frontend + backend | **Next.js (App Router)** | SSR = fast on cheap Android. One codebase. |
| DB | **SQLite → Postgres (Supabase)** | SQLite for local dev; Supabase free tier is enough for 100 gyms |
| Auth | **Phone OTP** (Supabase Auth / MSG91) | Gym owners will not remember a password |
| WhatsApp | `wa.me` v1 → Cloud API v2 | See §6 |
| Cron | **Vercel Cron** or a ₹300/mo VPS crontab | Daily 9am job: find expiring, send messages |
| Payments | **Razorpay** (subscriptions) | UPI Autopay mandates work out of the box |
| Hosting | Vercel free → ₹1,700/mo Pro | Or Hetzner VPS ₹500/mo if you want to save |

**Total running cost at 100 customers: roughly ₹3,000–6,000/month.** At ₹999 × 100 = ₹99,900 revenue, that's ~95% margin.

**Do not build a mobile app in v1.** A website works on the phone browser and you can ship fixes instantly.

---

## 9. Build plan

### Week 0 — Validation (do this first)
5 gym visits using `VALIDATION.md`. **Zero code until this is done.**

### Week 1 — Core
- [ ] DB schema + seed script
- [ ] Phone OTP login
- [ ] Dashboard screen (port `index.html` to React)
- [ ] Add member / bulk paste import
- [ ] `wa.me` send + log `last_reminded`

### Week 2 — The money loop
- [ ] Mark renewed (extends expiry, resets PT, logs revenue)
- [ ] Razorpay subscription + paywall
- [ ] Settings: gym name, custom message template
- [ ] Deploy to production

### Week 3 — Polish + first customers
- [ ] WhatsApp Cloud API template approval chased
- [ ] Onboard gym #1 and #2 **personally** — enter their member list yourself
- [ ] Weekly email digest to owner: "3 expiring, 2 lapsed, ₹5,400 at risk"
- [ ] PWA manifest so it installs to the home screen

**Ship at the end of week 2 even if it's rough.** Nothing you learn in week 3 is worth more than gym #1's feedback in week 2.

---

## 10. Metrics that decide whether this survives

| Metric | Target | Meaning |
|---|---|---|
| **Owner logs in ≥3×/week** | > 60% | Product is part of their routine |
| **WhatsApp send rate** | > 80% of due members | They trust the tool |
| **Renewal rate on reminded members** | > 40% | The pitch is true |
| **Month-2 retention** | > 85% | They see the value and keep paying |
| **Time to first "renewed" click** | < 10 min after onboarding | The aha moment lands |

**Kill criterion:** if fewer than half of your first 10 gyms still log in after 60 days, the problem isn't real enough. Stop and re-validate before adding features.

---

## 11. Risks

| Risk | Mitigation |
|---|---|
| **Owner never enters data** | You enter it for them. First 3 gyms: do it in person. |
| **WhatsApp template rejected by Meta** | Submit 3 variants on day 1. `wa.me` is the fallback. |
| **Members reply "stop messaging me"** | Add an opt-out flag on the member record. Respect it. |
| **Owner shares the login** | Fine at this scale. Charge per gym, not per seat. |
| **Free competitor appears** | Unlikely — the incumbents are paid and clunky. Your moat is that you call them back. |
| **DPDP Act compliance** | Store only name + phone. Consent checkbox at member creation. Deletion on request. Host in India (Vercel Mumbai / Supabase ap-south-1). |
| **Seasonal churn (Jan–Feb surge, monsoon dip)** | Expected. Price yearly and let cash flow absorb it. |

---

## 12. First 10 customers (the part that actually determines success)

Do not run ads. Do not build an SEO site yet.

1. **Google Maps → "gym near me" → 40 results in your city.** Build a sheet: name, owner name, phone, member count (often on the listing).
2. **Visit 8 per day, 11am–1pm.** No pitch, just the 5 questions from `VALIDATION.md`.
3. **For anyone who leans in:** "Give me your member register for 20 minutes, I'll set it up and show you. Free for 30 days."
4. **Set them up on the spot.** This is your entire onboarding — do it manually for the first 5 gyms and write down every friction point.
5. **Day 3 call:** "Did you send any reminders?" If no, your product is too hard. Fix it.
6. **Day 30:** convert to paid at ₹499 lifetime-locked. Ask for 2 referrals.
7. **Repeat until 10 paying.** Then, and only then, add automation, raise to ₹999, and start on the next city.

**Realistic timeline to ₹1 lakh/month: 4–8 months of persistent door-knocking.** Slower than an app, far more certain than one.

---

## 13. Naming and positioning

Working name: **Renewal Radar**. Alternatives: *ExpiryAlert*, *MemberPulse*, *GymKeeper*.

**Positioning line for the landing page:**

> "Your register can't tell you who's about to quit.
> Renewal Radar can — and it messages them for you."

**Do not call it "gym management software."** That category is crowded and the owner already thinks those tools are overpriced and complicated. You are a **renewal tool**.
