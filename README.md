# Renewal Radar — micro-SaaS for gyms

**[▶ Open the live demo](https://774krish.github.io/gym-renewal-/)**

A one-screen tool that shows a gym owner **who is about to expire, who already lapsed, and how much money is sitting in those two lists** — then sends the WhatsApp reminder in one tap.

Not gym management software. Not attendance. Not payroll. One job: **stop silent non-renewals.**

---

## The pitch (memorise this)

> "Recover two lapsed members a month and this app pays for itself."

A gym with 150 members charging £39/month has roughly **£5,850 of membership revenue expiring every month**. Every silently-lost member is £468/year gone. The owner currently tracks this in a paper register and hopes.

---

## What's in this folder

| File | Purpose |
|---|---|
| `index.html` | **Working demo.** Open it in a browser. Click around. Show it to gym owners while you ask questions. No install, no internet needed. |
| `VALIDATION.md` | The printable question sheet for visiting 5 gyms + decision rules. **Do this before writing any real code.** |
| `SPEC.md` | Full product spec: screens, data model, WhatsApp options, pricing, build plan. |

---

## Open the demo

Double-click `index.html`, or from PowerShell:

```powershell
Start-Process "c:\my portfolio\to do\gym-renewal-radar\index.html"
```

It loads 12 sample members, so it looks like a real gym on day one. Data is saved in your browser; "Reset demo data" restores the samples.

What to click when demoing:
1. Point at the red **"₹ at risk"** number. Say nothing. Let them read it.
2. Tap **WhatsApp** on a row — a pre-filled message opens.
3. Tap **Renew** — watch the stat counters move.

### What the demo can do

| Action | Where |
|---|---|
| **Set your own gym name** | Click the gym name under "Renewal Radar" |
| **Switch dark / light theme** | Sun/moon button, top right |
| **Sort by urgency / A–Z / Z–A** | Sort button beside the search box |
| **Flag a member to watch** | **☆ Flag** on any row — appears in the Flagged tab |
| **Add a note to a member** | **✎ Add note** on any row — the note shows on the card and is searchable |
| Search by name or phone number | Search box at the top |
| Add a member | **＋ Add member** |
| Edit name / number / plan / expiry / PT | **Edit** on any row |
| Bulk paste a member list | **⇪ Import list** |
| Mark someone as left the gym | **Mark as left** on any row |
| Bring them back | **Reactivate** (in the Left state) |
| Delete permanently | **Delete** on any row |
| Send the WhatsApp reminder | **💬 WhatsApp** |
| Record a renewal | **✓ Mark renewed** |

**Import format** — one member per line, comma/tab separated:
```
Rahul Sharma, 9876543210, monthly
Priya Nair, 9820123456, quarterly, 2026-08-15
Amit Kumar, 9812345678
```
Only name and 10-digit phone are required. Plan defaults to `monthly`; blank date means today.

**"Mark as left" vs "Delete"** — this distinction matters when you demo. *Left* keeps the history and removes them from your money-at-risk numbers. *Delete* is permanent and only for data-entry mistakes. Tell the owner that's the difference; it shows you thought about real use.

**Flagging** is the "keep an eye on this person" button. A flagged member gets a gold star, appears in the **Flagged** tab, and also shows up under **Needs action** even if their membership is healthy. Use it for the member who's been skipping weeks, the one who asked about cancelling, or the lead who's still deciding — the people who need a phone call rather than a reminder.

**Notes** hold the context a spreadsheet can't: *"asked about pausing in Nov,"* *"referred by Hannah — don't lose him,"* *"wants PT at renewal."* A note displays on the card (up to 3 lines, click to expand) and is included in search — so typing `pause` or `referral` finds the person even if you've forgotten their name. Ctrl/Cmd+Enter saves.

**The gym name is editable** — click it, type the real name, and it updates the header *and* every WhatsApp message. Set it to the actual gym you're standing in before you demo. The optional second field (area/location) shows after a dot.

---

## Before you build anything real

Go talk to 5 gym owners. `VALIDATION.md` has the script.

**Green light if:** 3 of 5 say they track expiry in a register/notebook or their head, and can name a member they lost without knowing why.
**Walk away if:** 3 of 5 already use an app they're happy with.

---

## If it validates

Stack in `SPEC.md`:
- **Next.js** (server-rendered — fast on cheap Android)
- **Postgres** via Supabase (or SQLite for v1)
- **WhatsApp Cloud API** for automated reminders (~₹0.10–0.30/message in India)
- Deploy on Vercel or a ₹500/month VPS

Timeline: **2–3 weeks** for v1 if the prototype is your spec.

Pricing: **₹999/month** for up to 200 members. ₹1,999 for unlimited + PT ledger.
Target: **100 gyms = ₹1 lakh/month.**

---

## What this is NOT

- Not a fitness app for members
- Not attendance biometrics
- Not diet plans, payroll, accounting, or inventory
- Not a mobile app (v1 is a website — the owner uses it at the desk, members get plain WhatsApp)

Every feature you add dilutes the pitch. The whole product is one screen.
