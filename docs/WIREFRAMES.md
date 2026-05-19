# Wireframes — Sigma Marketplace

**Version:** 1.0  
**Date:** 2026-05-19

---

## Mobile App (Technician)

---

### Screen 1: Login

```
┌─────────────────────────────┐
│                             │
│         SIGMA               │
│      Marketplace            │
│                             │
│  ┌───────────────────────┐  │
│  │  Email address        │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │  Password          👁  │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │      LOG IN           │  │
│  └───────────────────────┘  │
│                             │
│  Forgot password?           │
│                             │
└─────────────────────────────┘
```

---

### Screen 2: Job List (Home Tab)

```
┌─────────────────────────────┐
│ ≡  My Jobs          🔔  👤  │
├─────────────────────────────┤
│                             │
│  TODAY — Mon 19 May         │
│                             │
│  ┌───────────────────────┐  │
│  │ 🟡 IN PROGRESS        │  │
│  │ Site: Telkom Margonda │  │
│  │ SOP: LAN Installation │  │
│  │ 4/9 steps · Rp 85,000 │  │
│  │ Scheduled: 09:00      │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ 🔵 ASSIGNED           │  │
│  │ Site: Tower Ciputat   │  │
│  │ SOP: Cable Inspection │  │
│  │ 0/6 steps · Rp 60,000 │  │
│  │ Scheduled: 14:00      │  │
│  └───────────────────────┘  │
│                             │
│  TOMORROW                   │
│                             │
│  ┌───────────────────────┐  │
│  │ 🔵 ASSIGNED           │  │
│  │ Site: BTS Depok Timur │  │
│  │ SOP: Site Survey      │  │
│  │ 0/12 steps · Rp 120k  │  │
│  │ Scheduled: 08:00      │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ ✅ COMPLETED (2)      │  │
│  │ View past jobs →      │  │
│  └───────────────────────┘  │
│                             │
├──────┬──────┬──────┬────────┤
│ Jobs │      │Earn  │Profile │
│  🏠  │      │  💰  │  👤    │
└──────┴──────┴──────┴────────┘
```

---

### Screen 3: Job Detail

```
┌─────────────────────────────┐
│ ←  Job Detail               │
├─────────────────────────────┤
│                             │
│  Telkom Margonda            │
│  LAN Installation SOP       │
│                             │
│  📍 Jl. Margonda Raya No.5  │
│  🗓  Mon, 19 May 2026       │
│  ⏰  09:00 – Est. 12:00     │
│  👤  Assigned by: Rina W.   │
│                             │
│  ┌───────────────────────┐  │
│  │ Rp 85,000 total pay   │  │
│  │ ████████░░░░ 4/9 done │  │
│  └───────────────────────┘  │
│                             │
│  SOP STEPS                  │
│                             │
│  ✅ 1. Site arrival check-in│
│        Rp 5,000  ✓ Done    │
│                             │
│  ✅ 2. Equipment unboxing   │
│        Rp 5,000  ✓ Done    │
│                             │
│  ✅ 3. Cable route survey   │
│        Rp 10,000 ✓ Done    │
│                             │
│  ✅ 4. Patch panel install  │
│        Rp 15,000 ✓ Done    │
│                             │
│  ▶ 5. Switch rack mount     │ ← current
│       Rp 15,000  Tap to do │
│                             │
│  ○ 6. Cable termination     │
│       Rp 15,000  Locked    │
│                             │
│  ○ 7. Cable labeling        │
│       Rp 10,000  Locked    │
│                             │
│  ○ 8. Network testing       │
│       Rp 10,000  Locked    │
│                             │
│  ○ 9. Site sign-off         │
│       Rp 0 (required)       │
│                             │
└─────────────────────────────┘
```

---

### Screen 4: Task Execution

```
┌─────────────────────────────┐
│ ←  Step 5 of 9              │
├─────────────────────────────┤
│                             │
│  Switch Rack Mount          │
│  Rp 15,000                  │
│                             │
│  INSTRUCTIONS               │
│  ┌───────────────────────┐  │
│  │Mount the 24-port switch│  │
│  │into rack U6. Secure   │  │
│  │with 4 screws. Attach  │  │
│  │power cable.           │  │
│  └───────────────────────┘  │
│                             │
│  PHOTO EVIDENCE (required)  │
│                             │
│  ┌──────┐ ┌──────┐ ┌─────┐ │
│  │      │ │      │ │  +  │ │
│  │ 📷   │ │ 📷   │ │ Add │ │
│  │      │ │      │ │     │ │
│  └──────┘ └──────┘ └─────┘ │
│   Tap to view   Add photo   │
│                             │
│  NOTES (optional)           │
│  ┌───────────────────────┐  │
│  │ Add note or exception  │  │
│  │                        │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │   MARK AS COMPLETE    │  │
│  └───────────────────────┘  │
│                             │
│  ⚠  No internet — will sync │
│     when connected          │
└─────────────────────────────┘
```

---

### Screen 5: Earnings

```
┌─────────────────────────────┐
│ ≡  Earnings             👤  │
├─────────────────────────────┤
│                             │
│  May 2026                   │
│  ┌───────────────────────┐  │
│  │ Total Earned          │  │
│  │ Rp 1,245,000          │  │
│  │                       │  │
│  │ ████████████████░░░░  │  │
│  │ 18 of 22 working days  │  │
│  └───────────────────────┘  │
│                             │
│  This Week                  │
│  ┌───────────────────────┐  │
│  │ Rp 315,000            │  │
│  │ 4 jobs · 31 steps     │  │
│  └───────────────────────┘  │
│                             │
│  JOB BREAKDOWN              │
│                             │
│  ┌───────────────────────┐  │
│  │ Telkom Margonda       │  │
│  │ 19 May · 9/9 steps    │  │
│  │               Rp 85k  │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ Tower Ciputat         │  │
│  │ 19 May · 6/6 steps    │  │
│  │               Rp 60k  │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ BTS Depok Timur       │  │
│  │ 18 May · 12/12 steps  │  │
│  │              Rp 120k  │  │
│  └───────────────────────┘  │
│                             │
├──────┬──────┬──────┬────────┤
│ Jobs │      │Earn  │Profile │
│  🏠  │      │  💰  │  👤    │
└──────┴──────┴──────┴────────┘
```

---

## Web Dashboard (Client Admin)

---

### Screen 6: Dashboard Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Technicians  SOPs  Payouts               │
│         │                                              Rina W. ▾     │
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Good morning, Rina!  Monday, 19 May 2026                           │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────┐ │
│  │ Active Jobs  │  │ Technicians  │  │Tasks Done    │  │ Payout  │ │
│  │              │  │    Today     │  │  Today       │  │ This Mo │ │
│  │     12       │  │    8 / 14   │  │   47 / 63    │  │ Rp 4.2M │ │
│  │  ↑ 3 new     │  │  on-site    │  │   74%        │  │         │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └─────────┘ │
│                                                                      │
│  LIVE JOB MAP                          ACTIVE JOBS                  │
│  ┌─────────────────────────────┐  ┌──────────────────────────────┐  │
│  │                             │  │  Site              Tech  Prog │  │
│  │   🗺  [Map placeholder]     │  │  ──────────────────────────── │  │
│  │      📍 📍   📍            │  │  Telkom Margonda   Budi  4/9  │  │
│  │         📍       📍        │  │  Tower Ciputat     Andi  0/6  │  │
│  │                             │  │  BTS Depok         Reza  12/12│  │
│  │                             │  │  Site Bogor        Hadi  2/8  │  │
│  └─────────────────────────────┘  └──────────────────────────────┘  │
│                                                                      │
│  RECENT ACTIVITY                                                     │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │ 10:42  Budi completed step "Patch panel install" — Telkom      │ │
│  │ 10:31  Reza completed job — BTS Depok Timur ✅                 │ │
│  │ 09:15  Andi checked in — Tower Ciputat                         │ │
│  │ 09:00  3 new jobs dispatched for today                         │ │
│  └────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 7: Job Orders List

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Technicians  SOPs  Payouts               │
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Job Orders                                    [+ New Job Order]     │
│                                                                      │
│  Filters: Status ▾  Technician ▾  Date Range ▾  SOP ▾   [Search]   │
│                                                                      │
│  ┌────┬─────────────────┬──────────┬──────────┬────────┬──────────┐ │
│  │ #  │ Site            │Technician│ SOP      │ Status │ Payout   │ │
│  ├────┼─────────────────┼──────────┼──────────┼────────┼──────────┤ │
│  │ 83 │ Telkom Margonda │ Budi S.  │ LAN Inst │🟡 4/9  │ Rp 85k   │ │
│  │ 82 │ Tower Ciputat   │ Andi R.  │ Cable Ins│🔵 0/6  │ Rp 60k   │ │
│  │ 81 │ BTS Depok Timur │ Reza M.  │ Survey   │✅ Done │ Rp 120k  │ │
│  │ 80 │ Site Bogor      │ Hadi P.  │ LAN Inst │🟡 2/8  │ Rp 80k   │ │
│  │ 79 │ Gedung Plaza    │ Fajar K. │ Fiber OFC│✅ Done │ Rp 200k  │ │
│  │ 78 │ RS Fatmawati    │ Budi S.  │ LAN Inst │✅ Done │ Rp 85k   │ │
│  └────┴─────────────────┴──────────┴──────────┴────────┴──────────┘ │
│                                                                      │
│  Showing 6 of 83 jobs  ←  1  2  3 ... 14  →                        │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 8: Create Job Order

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Technicians  SOPs  Payouts               │
├─────────┴────────────────────────────────────────────────────────────┤
│  ←  Create Job Order                                                 │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SITE INFORMATION                                                    │
│  ┌──────────────────────────┐   ┌─────────────────────────────────┐ │
│  │ Site Name *              │   │ Address *                       │ │
│  │ e.g. Telkom Margonda     │   │ Full address or coordinates     │ │
│  └──────────────────────────┘   └─────────────────────────────────┘ │
│                                                                      │
│  ASSIGNMENT                                                          │
│  ┌──────────────────────────┐   ┌─────────────────────────────────┐ │
│  │ Technician *          ▾  │   │ Scheduled Date & Time *         │ │
│  │ Select technician        │   │ 19 May 2026  09:00              │ │
│  └──────────────────────────┘   └─────────────────────────────────┘ │
│                                                                      │
│  SOP TEMPLATE                                                        │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Select SOP *                                              ▾  │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  [Selected: LAN Installation]                                        │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  9 steps  ·  Est. 3h  ·  Total payout: Rp 85,000           │   │
│  │  1. Site arrival check-in              Rp  5,000             │   │
│  │  2. Equipment unboxing                 Rp  5,000             │   │
│  │  3. Cable route survey                 Rp 10,000             │   │
│  │  4. Patch panel install                Rp 15,000             │   │
│  │  5. Switch rack mount                  Rp 15,000             │   │
│  │  ... +4 more steps                                           │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  NOTES (optional)                                                    │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Special instructions for this job                            │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│              [Cancel]         [Create & Dispatch Job]                │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 9: Job Detail (Real-time Progress)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  ← Jobs  /  Job #83 — Telkom Margonda                     │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LAN Installation · Budi Santoso                    🟡 IN PROGRESS  │
│  📍 Jl. Margonda Raya No.5  ·  🗓 19 May 2026  ·  ⏰ 09:00        │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ Progress     │  │ Elapsed Time │  │ Payout Earned│              │
│  │  4 / 9 steps │  │   1h 23m     │  │   Rp 35,000  │              │
│  │ ████████░░░░ │  │              │  │  / Rp 85,000  │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                      │
│  SOP STEP PROGRESS                         EVIDENCE PHOTOS          │
│  ┌────────────────────────────────┐  ┌───────────────────────────┐  │
│  │ ✅ 1. Site arrival check-in    │  │                           │  │
│  │       09:02  ·  1 photo        │  │  [Photo grid — click to  │  │
│  │ ✅ 2. Equipment unboxing       │  │   view full size]         │  │
│  │       09:18  ·  2 photos       │  │                           │  │
│  │ ✅ 3. Cable route survey       │  │  Step 1: ▓▓ 1 photo      │  │
│  │       09:41  ·  3 photos       │  │  Step 2: ▓▓ 2 photos     │  │
│  │ ✅ 4. Patch panel install      │  │  Step 3: ▓▓ 3 photos     │  │
│  │       10:24  ·  2 photos       │  │  Step 4: ▓▓ 2 photos     │  │
│  │ ▶ 5. Switch rack mount         │  │                           │  │
│  │       In progress...           │  │                           │  │
│  │ ○ 6. Cable termination         │  └───────────────────────────┘  │
│  │ ○ 7. Cable labeling            │                                  │
│  │ ○ 8. Network testing           │  ACTIVITY LOG                   │
│  │ ○ 9. Site sign-off             │  ┌───────────────────────────┐  │
│  └────────────────────────────────┘  │ 10:42 Step 4 completed   │  │
│                                       │ 09:41 Step 3 completed   │  │
│                                       │ 09:18 Step 2 completed   │  │
│                                       │ 09:02 Job started        │  │
│                                       └───────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 10: SOP Builder

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Technicians  SOPs  Payouts               │
├─────────┴────────────────────────────────────────────────────────────┤
│  ←  SOP Builder  /  LAN Installation  v1.2          [Save] [Publish] │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────┐   ┌─────────────────────────────────┐ │
│  │ SOP Name *               │   │ Estimated Duration              │ │
│  │ LAN Installation         │   │  3 hours                        │ │
│  └──────────────────────────┘   └─────────────────────────────────┘ │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Description                                                  │   │
│  │ Standard procedure for LAN installation at client site...   │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  STEPS                                              Total: Rp 85,000 │
│                                                                      │
│  ⋮ ┌──────────────────────────────────────────┬──────────┬───────┐  │
│    │ 1. Site arrival check-in                  │ Rp 5,000 │ ✏️ 🗑 │  │
│    │    📷 Photo required  📝 Notes optional    │          │      │  │
│    └──────────────────────────────────────────┴──────────┴───────┘  │
│                                                                      │
│  ⋮ ┌──────────────────────────────────────────┬──────────┬───────┐  │
│    │ 2. Equipment unboxing                     │ Rp 5,000 │ ✏️ 🗑 │  │
│    │    📷 Photo required  📝 Notes optional    │          │      │  │
│    └──────────────────────────────────────────┴──────────┴───────┘  │
│                                                                      │
│  ⋮ ┌──────────────────────────────────────────┬──────────┬───────┐  │
│    │ 3. Cable route survey                     │Rp 10,000 │ ✏️ 🗑 │  │
│    │    📷 Photo required  ⚠️ Mandatory order  │          │      │  │
│    └──────────────────────────────────────────┴──────────┴───────┘  │
│                                                                      │
│                              [+ Add Step]                            │
│                                                                      │
│  ⚠  This SOP is used in 3 active jobs.                              │
│     Publishing will create v1.3 — existing jobs keep v1.2.          │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 11: Payout Report

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Technicians  SOPs  Payouts               │
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Payout Report                   Period: May 2026 ▾  [Export CSV]   │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ Total Payout │  │ Jobs Done    │  │ Steps Done   │              │
│  │  Rp 4,240,000│  │     38       │  │    342       │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                      │
│  ┌────┬─────────────────┬──────────┬─────────┬────────┬──────────┐ │
│  │    │ Technician      │ Jobs     │ Steps   │ Amount │ Status   │ │
│  ├────┼─────────────────┼──────────┼─────────┼────────┼──────────┤ │
│  │ ▼  │ Budi Santoso    │    9     │   81    │ Rp 765k│ ⏳ Pending│ │
│  │    │   └ View jobs   │          │         │        │ [Mark Paid│ │
│  │ ▼  │ Andi Raharjo    │    7     │   63    │ Rp 595k│ ✅ Paid   │ │
│  │ ▼  │ Reza Maulana    │    8     │   72    │ Rp 680k│ ⏳ Pending│ │
│  │ ▼  │ Hadi Pratama    │    6     │   54    │ Rp 510k│ ✅ Paid   │ │
│  │ ▼  │ Fajar Kurnia    │    8     │   72    │ Rp 690k│ ⏳ Pending│ │
│  └────┴─────────────────┴──────────┴─────────┴────────┴──────────┘ │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 12: Technician Management

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Technicians  SOPs  Payouts               │
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Technicians                                  [+ Invite Technician]  │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  🔍 Search by name or email                                  │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  👤 Budi Santoso          🟢 Active · On-site                  │ │
│  │     budi@email.com · 9 jobs this month · Rp 765k earned       │ │
│  │                                    [View]  [Deactivate]       │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │  👤 Andi Raharjo          🟢 Active · Available               │ │
│  │     andi@email.com · 7 jobs this month · Rp 595k earned       │ │
│  │                                    [View]  [Deactivate]       │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │  👤 Reza Maulana          🟢 Active · On-site                  │ │
│  │     reza@email.com · 8 jobs this month · Rp 680k earned       │ │
│  │                                    [View]  [Deactivate]       │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │  👤 Siti Nurhaliza        ⚪ Inactive                          │ │
│  │     siti@email.com · Last active: 2 Apr 2026                  │ │
│  │                                   [View]  [Reactivate]        │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  14 technicians total  ·  11 active  ·  3 inactive                  │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Notification Center (Mobile)

```
┌─────────────────────────────┐
│ ←  Notifications            │
├─────────────────────────────┤
│                             │
│  TODAY                      │
│                             │
│  ┌───────────────────────┐  │
│  │ 🔵 New job assigned   │  │
│  │ BTS Depok Timur —     │  │
│  │ tomorrow 08:00        │  │
│  │              10:42 AM │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ 💰 Payout processed   │  │
│  │ April 2026 payout:    │  │
│  │ Rp 3,450,000          │  │
│  │               9:00 AM │  │
│  └───────────────────────┘  │
│                             │
│  YESTERDAY                  │
│                             │
│  ┌───────────────────────┐  │
│  │ ✅ Job completed      │  │
│  │ BTS Depok confirmed   │  │
│  │ by Rina W.            │  │
│  │               6:30 PM │  │
│  └───────────────────────┘  │
│                             │
└─────────────────────────────┘
```
