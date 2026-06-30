# Wireframes — Sigma Marketplace

**Version:** 1.1  
**Date:** 2026-05-19

> **v2.0 Projects wireframes (HTML):** the engagement-model screens — project builder (4 pricing models),
> client monitor, vendor reporting, worker management, and accrual payouts — are clickable mockups in
> [`/wireframes/index.html`](../wireframes/index.html) (styled with the Sigma ui-kit). The ASCII screens below are
> the original v1.1 job-order flows.

---

## Mobile App (Worker — belongs to Vendor)

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
│  │ Telkom Margonda       │  │
│  │ LAN Installation      │  │
│  │ 4 / 9 steps done      │  │
│  │ Scheduled: 09:00      │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ 🔵 ASSIGNED           │  │
│  │ Tower Ciputat         │  │
│  │ Cable Inspection      │  │
│  │ 0 / 6 steps done      │  │
│  │ Scheduled: 14:00      │  │
│  └───────────────────────┘  │
│                             │
│  TOMORROW                   │
│                             │
│  ┌───────────────────────┐  │
│  │ 🔵 ASSIGNED           │  │
│  │ BTS Depok Timur       │  │
│  │ Site Survey           │  │
│  │ 0 / 12 steps done     │  │
│  │ Scheduled: 08:00      │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ ✅ COMPLETED (2)      │  │
│  │ View past jobs →      │  │
│  └───────────────────────┘  │
│                             │
├──────┬──────┬──────┬────────┤
│ Jobs │      │History│Profile│
│  🏠  │      │  📋   │  👤   │
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
│  LAN Installation           │
│                             │
│  📍 Jl. Margonda Raya No.5  │
│  🗓  Mon, 19 May 2026       │
│  ⏰  09:00 – Est. 12:00     │
│  🏢  Client: PT. Telkom     │
│                             │
│  ┌───────────────────────┐  │
│  │ ████████░░░░ 4/9 done │  │
│  └───────────────────────┘  │
│                             │
│  SOP STEPS                  │
│                             │
│  ✅ 1. Site arrival check-in│
│        ✓ Done               │
│                             │
│  ✅ 2. Equipment unboxing   │
│        ✓ Done               │
│                             │
│  ✅ 3. Cable route survey   │
│        ✓ Done               │
│                             │
│  ✅ 4. Patch panel install  │
│        ✓ Done               │
│                             │
│  ▶ 5. Switch rack mount     │ ← current
│       Tap to start          │
│                             │
│  ○ 6. Cable termination     │
│       Locked                │
│                             │
│  ○ 7. Cable labeling        │
│       Locked                │
│                             │
│  ○ 8. Network testing       │
│       Locked                │
│                             │
│  ○ 9. Site sign-off         │
│       Required              │
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
│  Step 5 of 9                │
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

### Screen 5: Job History

```
┌─────────────────────────────┐
│ ≡  Job History          👤  │
├─────────────────────────────┤
│                             │
│  May 2026                   │
│  ┌───────────────────────┐  │
│  │ 18 jobs completed     │  │
│  │ 342 steps done        │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ ✅ Telkom Margonda    │  │
│  │ LAN Installation      │  │
│  │ 19 May · 9/9 steps    │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ ✅ Tower Ciputat      │  │
│  │ Cable Inspection      │  │
│  │ 19 May · 6/6 steps    │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ ✅ BTS Depok Timur    │  │
│  │ Site Survey           │  │
│  │ 18 May · 12/12 steps  │  │
│  └───────────────────────┘  │
│                             │
├──────┬──────┬──────┬────────┤
│ Jobs │      │History│Profile│
│  🏠  │      │  📋   │  👤   │
└──────┴──────┴──────┴────────┘
```

---

## Web Dashboard — Client Admin

---

### Screen 6: Client Dashboard Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Vendors  SOPs                            │
│         │                                         PT. Telkom · Rina ▾│
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Good morning, Rina!  Monday, 19 May 2026                           │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────┐ │
│  │ Active Jobs  │  │ Workers  │  │ Steps Done   │  │Pending  │ │
│  │              │  │  On-site     │  │   Today      │  │Payouts  │ │
│  │     12       │  │     8        │  │  47 / 63     │  │   9     │ │
│  │  ↑ 3 new     │  │              │  │   74%        │  │         │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └─────────┘ │
│                                                                      │
│  LIVE JOB MAP                          ACTIVE JOBS                  │
│  ┌─────────────────────────────┐  ┌──────────────────────────────┐  │
│  │                             │  │  Site         Vendor   Prog  │  │
│  │   🗺  [Map placeholder]     │  │  ──────────────────────────── │  │
│  │      📍 📍   📍            │  │  Telkom Marg  Sigma T  4/9   │  │
│  │         📍       📍        │  │  Tower Ciput  Sigma T  0/6   │  │
│  │                             │  │  BTS Depok    NetPro   12/12 │  │
│  └─────────────────────────────┘  └──────────────────────────────┘  │
│                                                                      │
│  RECENT ACTIVITY                                                     │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │ 10:42  Budi (Sigma Teknik) completed step 4 — Telkom Margonda  │ │
│  │ 10:31  Reza (NetPro) completed job — BTS Depok Timur ✅        │ │
│  │ 09:15  Andi (Sigma Teknik) checked in — Tower Ciputat          │ │
│  └────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 7: Job Orders List (Client)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Vendors  SOPs                            │
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Job Orders                                    [+ New Job Order]     │
│                                                                      │
│  Status ▾  Vendor ▾  Service Category ▾  Date Range ▾  [Search]    │
│                                                                      │
│  ┌────┬─────────────────┬──────────────┬──────────────┬───────────┐ │
│  │ #  │ Site            │ Vendor · Tech│ Category     │ Status    │ │
│  ├────┼─────────────────┼──────────────┼──────────────┼───────────┤ │
│  │ 83 │ Telkom Margonda │Sigma T · Budi│ LAN Install  │🟡 4/9     │ │
│  │ 82 │ Tower Ciputat   │Sigma T · Andi│ Cable Insp   │🔵 0/6     │ │
│  │ 81 │ BTS Depok Timur │NetPro · Reza │ Site Survey  │✅ Done    │ │
│  │ 80 │ Site Bogor      │Sigma T · Hadi│ LAN Install  │🟡 2/8     │ │
│  │ 79 │ Gedung Plaza    │NetPro · Fajar│ Fiber Optic  │✅ Done    │ │
│  └────┴─────────────────┴──────────────┴──────────────┴───────────┘ │
│                                                                      │
│  Showing 5 of 83 jobs  ←  1  2  3 ... 17  →                        │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 8: Create Job Order (Client)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Vendors  SOPs                            │
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
│  SERVICE                                                             │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Service Category *                                        ▾  │   │
│  │ e.g. LAN Installation, Fiber Optic Splicing, Site Survey     │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  [Selected: LAN Installation]                                        │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  SOP: LAN Installation v1.2  ·  9 steps  ·  Est. 3h         │   │
│  │  1. Site arrival check-in                                    │   │
│  │  2. Equipment unboxing                                       │   │
│  │  3. Cable route survey                                       │   │
│  │  ... +6 more steps                          [View full SOP]  │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ASSIGNMENT                                                          │
│  ┌──────────────────────────┐   ┌─────────────────────────────────┐ │
│  │ Vendor *              ▾  │   │ Worker *               ▾   │ │
│  │ Select partner vendor    │   │ Select from vendor's team      │ │
│  └──────────────────────────┘   └─────────────────────────────────┘ │
│                                                                      │
│  ┌──────────────────────────┐   ┌─────────────────────────────────┐ │
│  │ Scheduled Date & Time *  │   │ Payout Amount (to vendor) *    │ │
│  │ 19 May 2026  09:00       │   │ Rp _______________             │ │
│  └──────────────────────────┘   └─────────────────────────────────┘ │
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

### Screen 9: Job Detail — Real-time Progress (Client)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  ← Jobs  /  Job #83 — Telkom Margonda                     │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LAN Installation                                   🟡 IN PROGRESS  │
│  📍 Jl. Margonda Raya No.5  ·  🗓 19 May 2026  ·  ⏰ 09:00        │
│  🏢 Vendor: PT. Sigma Teknik  ·  👤 Budi Santoso                   │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ Progress     │  │ Elapsed Time │  │ Payout       │              │
│  │  4 / 9 steps │  │   1h 23m     │  │  Rp 350,000  │              │
│  │ ████████░░░░ │  │              │  │  Pending     │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                      │
│  SOP STEP PROGRESS                         EVIDENCE PHOTOS          │
│  ┌────────────────────────────────┐  ┌───────────────────────────┐  │
│  │ ✅ 1. Site arrival check-in    │  │                           │  │
│  │       09:02  ·  1 photo        │  │  Step 1: ▓▓ 1 photo      │  │
│  │ ✅ 2. Equipment unboxing       │  │  Step 2: ▓▓ 2 photos     │  │
│  │       09:18  ·  2 photos       │  │  Step 3: ▓▓ 3 photos     │  │
│  │ ✅ 3. Cable route survey       │  │  Step 4: ▓▓ 2 photos     │  │
│  │       09:41  ·  3 photos       │  │                           │  │
│  │ ✅ 4. Patch panel install      │  │  [Click to view gallery]  │  │
│  │       10:24  ·  2 photos       │  │                           │  │
│  │ ▶ 5. Switch rack mount         │  └───────────────────────────┘  │
│  │       In progress...           │                                  │
│  │ ○ 6. Cable termination         │  ACTIVITY LOG                   │
│  │ ○ 7. Cable labeling            │  ┌───────────────────────────┐  │
│  │ ○ 8. Network testing           │  │ 10:42 Step 4 completed   │  │
│  │ ○ 9. Site sign-off             │  │ 09:41 Step 3 completed   │  │
│  └────────────────────────────────┘  │ 09:02 Job started        │  │
│                                       └───────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 10: Vendor Management (Client)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Vendors  SOPs                            │
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Partner Vendors                                                     │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  🏢 PT. Sigma Teknik               🟢 Active                   │ │
│  │     12 workers  ·  38 jobs this month                      │ │
│  │     Categories: LAN Install, Cable Inspection, Site Survey     │ │
│  │                                              [View]  [Jobs]   │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │  🏢 NetPro Solutions               🟢 Active                   │ │
│  │     8 workers  ·  21 jobs this month                       │ │
│  │     Categories: Fiber Optic, Site Survey                       │ │
│  │                                              [View]  [Jobs]   │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 11: SOP Viewer (Client — read-only)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Vendors  SOPs                            │
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  SOPs by Service Category                                           │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ 🔧 LAN Installation            v1.2  ·  9 steps  ·  Est. 3h  │   │
│  │    1. Site arrival check-in                                  │   │
│  │    2. Equipment unboxing                                     │   │
│  │    3. Cable route survey         [▼ View all steps]          │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ 🔧 Fiber Optic Splicing         v2.0  ·  14 steps  ·  Est 5h │   │
│  │    1. Safety equipment check                                 │   │
│  │    2. Cable preparation                                      │   │
│  │    3. Splice machine setup       [▼ View all steps]          │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ 🔧 Site Survey                  v1.0  ·  12 steps  ·  Est 4h │   │
│  │    1. Site arrival and registration                          │   │
│  │    2. Area perimeter walk        [▼ View all steps]          │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 12: Payout Approval (Client)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Vendors  SOPs  Payouts                   │
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Payouts                         Period: May 2026 ▾  [Export CSV]   │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ Total Payout │  │ Jobs Done    │  │ Pending      │              │
│  │  Rp 12.4M    │  │     38       │  │  Rp 7.2M     │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                      │
│  ┌─────────┬──────────────────┬──────────┬──────────┬────────────┐ │
│  │ Vendor  │ Job              │ Tech     │ Amount   │ Status     │ │
│  ├─────────┼──────────────────┼──────────┼──────────┼────────────┤ │
│  │Sigma T  │ Telkom Margonda  │ Budi S.  │ Rp 350k  │⏳ [Approve]│ │
│  │Sigma T  │ Tower Ciputat    │ Andi R.  │ Rp 250k  │⏳ [Approve]│ │
│  │NetPro   │ BTS Depok Timur  │ Reza M.  │ Rp 400k  │✅ Approved │ │
│  │Sigma T  │ Site Bogor       │ Hadi P.  │ Rp 300k  │⏳ [Approve]│ │
│  │NetPro   │ Gedung Plaza     │ Fajar K. │ Rp 500k  │✅ Approved │ │
│  └─────────┴──────────────────┴──────────┴──────────┴────────────┘ │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Web Dashboard — Vendor Admin

---

### Screen 13: Vendor Dashboard Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Workers  Payouts                     │
│         │                                    PT. Sigma Teknik · Andi ▾│
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Good morning, Andi!  Monday, 19 May 2026                           │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────┐ │
│  │ Active Jobs  │  │ Workers  │  │  Steps Done  │  │ Pending │ │
│  │ (all clients)│  │  On-site     │  │    Today     │  │Payouts  │ │
│  │     12       │  │    8 / 15    │  │  47 / 63     │  │ Rp 4.2M │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └─────────┘ │
│                                                                      │
│  TECHNICIAN STATUS                    TODAY'S JOBS                   │
│  ┌──────────────────────────┐  ┌──────────────────────────────────┐ │
│  │ Budi S.     🟡 On-site  │  │ Tech    Site           Progress  │ │
│  │ Andi R.     🟡 On-site  │  │ Budi    Telkom Marg    4/9       │ │
│  │ Reza M.     ✅ Done     │  │ Andi    Tower Ciputat  0/6       │ │
│  │ Hadi P.     🟡 On-site  │  │ Hadi    Site Bogor     2/8       │ │
│  │ Fajar K.    🔵 Assigned │  │ Fajar   Gedung Plaza   Pending   │ │
│  │ Siti N.     ⚪ Free     │  │                                  │ │
│  └──────────────────────────┘  └──────────────────────────────────┘ │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 14: Worker Management (Vendor)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Workers  Payouts                     │
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Workers                                  [+ Invite Worker]  │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  👤 Budi Santoso          🟢 Active · On-site                  │ │
│  │     budi@email.com · 9 jobs this month                         │ │
│  │                                    [View]  [Deactivate]       │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │  👤 Andi Raharjo          🟢 Active · On-site                  │ │
│  │     andi@email.com · 7 jobs this month                         │ │
│  │                                    [View]  [Deactivate]       │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │  👤 Siti Nurhaliza        ⚪ Inactive                          │ │
│  │     siti@email.com · Last active: 2 Apr 2026                  │ │
│  │                                   [View]  [Reactivate]        │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  15 workers total  ·  12 active  ·  3 inactive                  │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Screen 15: Payout Summary (Vendor)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA  │  Dashboard  Jobs  Workers  Payouts                     │
├─────────┴────────────────────────────────────────────────────────────┤
│                                                                      │
│  Payouts                         Period: May 2026 ▾  [Export CSV]   │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ Total Earned │  │  Approved    │  │  Pending     │              │
│  │  Rp 8.6M     │  │  Rp 4.4M    │  │  Rp 4.2M     │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                      │
│  ┌──────────────────┬──────────────┬──────────┬────────┬──────────┐ │
│  │ Job              │ Client       │ Tech     │ Amount │ Status   │ │
│  ├──────────────────┼──────────────┼──────────┼────────┼──────────┤ │
│  │ Telkom Margonda  │ PT. Telkom   │ Budi S.  │ Rp 350k│⏳ Pending│ │
│  │ Tower Ciputat    │ PT. Telkom   │ Andi R.  │ Rp 250k│⏳ Pending│ │
│  │ BTS Depok Timur  │ PT. Telkom   │ Reza M.  │ Rp 400k│✅ Approved│ │
│  │ Site Semarang    │ Indosat      │ Hadi P.  │ Rp 300k│✅ Approved│ │
│  └──────────────────┴──────────────┴──────────┴────────┴──────────┘ │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Super Admin — SOP Builder

---

### Screen 16: SOP Builder (Super Admin)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIGMA ADMIN  │  Orgs  Service Categories  SOPs  Payouts             │
├───────────────┴──────────────────────────────────────────────────────┤
│  ←  SOP Builder  /  LAN Installation  v1.2       [Save] [Publish]   │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────┐   ┌─────────────────────────────────┐ │
│  │ SOP Name *               │   │ Service Category *              │ │
│  │ LAN Installation         │   │ LAN Installation           ▾   │ │
│  └──────────────────────────┘   └─────────────────────────────────┘ │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Description                                                  │   │
│  │ Standard procedure for LAN installation at client site       │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  STEPS                                                               │
│                                                                      │
│  ⋮ ┌─────────────────────────────────────────────────────┬───────┐  │
│    │ 1. Site arrival check-in                             │ ✏️ 🗑 │  │
│    │    📷 Photo required  📝 Notes optional              │      │  │
│    └─────────────────────────────────────────────────────┴───────┘  │
│                                                                      │
│  ⋮ ┌─────────────────────────────────────────────────────┬───────┐  │
│    │ 2. Equipment unboxing                                │ ✏️ 🗑 │  │
│    │    📷 Photo required  📝 Notes optional              │      │  │
│    └─────────────────────────────────────────────────────┴───────┘  │
│                                                                      │
│  ⋮ ┌─────────────────────────────────────────────────────┬───────┐  │
│    │ 3. Cable route survey                                │ ✏️ 🗑 │  │
│    │    📷 Photo required  ⚠️ Mandatory order             │      │  │
│    └─────────────────────────────────────────────────────┴───────┘  │
│                                                                      │
│                              [+ Add Step]                            │
│                                                                      │
│  ⚠  This SOP is used in 3 active jobs.                              │
│     Publishing creates v1.3 — existing jobs keep v1.2.              │
│                                                                      │
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
│  │ by PT. Telkom         │  │
│  │ tomorrow 08:00        │  │
│  │              10:42 AM │  │
│  └───────────────────────┘  │
│                             │
│  ┌───────────────────────┐  │
│  │ ⏰ Reminder           │  │
│  │ Tower Ciputat starts  │  │
│  │ in 1 hour (14:00)     │  │
│  │              13:00 PM │  │
│  └───────────────────────┘  │
│                             │
│  YESTERDAY                  │
│                             │
│  ┌───────────────────────┐  │
│  │ ✅ Job confirmed      │  │
│  │ BTS Depok payout      │  │
│  │ approved by PT. Telkom│  │
│  │               6:30 PM │  │
│  └───────────────────────┘  │
│                             │
└─────────────────────────────┘
```
