# Sudut Silat — Project Plan

> Last updated: 2026-05-03 (rev 5 — scoring roles made symmetric: Juri 1 & Juri 2 can score either corner; dual-confirmation now matches corner + technique)
> Status: Pre-development / Planning

---

## 1. Background

**Sudut Silat** is a real-time scoring application designed for Pencak Silat (Tanding category) judges. The app digitises the scoring process across three judging roles and enforces the official PERSILAT scoring rules — including the dual-confirmation window mechanic to validate strike points.

---

## 2. Pencak Silat Tanding — Rules Summary

### 2.1 Match Format
- 3 rounds × 2 minutes, with 1-minute rest between rounds
- Two athletes: **Sudut Merah** (Red Corner) vs **Sudut Biru** (Blue Corner)
- Winner declared if they win 2 consecutive rounds; otherwise points from all rounds are accumulated

### 2.2 Valid Scoring Area (TOGOK)
The legal target zone is the **trunk (togok)**:
- Chest, abdomen (navel and above), left/right ribs, back of the trunk
- **Excluded**: neck upward (head), below navel to groin
- Limbs accidentally hit = no score, no penalty

### 2.3 Scoring Points

| Technique | Points |
|---|---|
| Hand strike (clean, no block) | 1 pt |
| Evade/parry + hand counter | 1 + 1 = 2 pts |
| Kick (clean, no block) | 2 pts |
| Evade/parry + kick counter | 1 + 2 = 3 pts |
| Takedown / floor the opponent | 3 pts |
| Evade/parry + takedown | 1 + 3 = 4 pts |
| **Pencak Point** (subjective, end of round) | 1 pt (juri discretion) |

### 2.4 Violations & Penalties

**Pelanggaran Ringan (Light Violations)**
- Excessive shouting
- Turning back on opponent
- Stepping out of the arena
- Incorrect attack trajectory
- Lying/sweeping to waste time

**Pelanggaran Berat (Heavy Violations)**
- Attacking illegal body parts (head, neck, groin)
- Attempting to break joints directly
- Headbutt
- Attacking before "mulai" or after "stop"
- Grappling, biting, scratching, gripping, hair-pulling
- Deliberate throw out of arena

**Punishment & Point Deductions**

| Level | Trigger | Deduction |
|---|---|---|
| Teguran (Warning) | First light violation | 0 pts |
| Peringatan I | Repeated light / first heavy | −1 pt |
| Peringatan II | Continued violation | −2 pts |
| Hukuman I | Accumulation | −5 pts |
| Hukuman II / DQ | Severe / repeated heavy | −10 pts / Disqualification |

### 2.5 Judging Panel (Official)
Officially: 1 Referee (Wasit) + 5 Judges (Juri)

**For this app (simplified/local version):**
- **Juri 1** — scores valid hits for **either** corner (Merah or Biru); corner is selected per entry, not fixed at role selection
- **Juri 2** — scores valid hits for **either** corner (Merah or Biru); same mechanic as Juri 1
- **Juri Pelanggaran** — records fouls and applies point deductions to either athlete

> Both Juri 1 and Juri 2 are **symmetric**: they observe the same match and each independently decides which corner scored on each exchange. The corner is not fixed per device — it is chosen in the scoring flow for every individual entry.

---

## 3. Platform Decision

### Recommendation: **Progressive Web App (PWA)**

| Criteria | PWA | Native Mobile | Desktop App |
|---|---|---|---|
| No paid hosting needed | ✅ Local network / LAN | ✅ | ✅ |
| Works on any device | ✅ Browser-based | ❌ Per-platform build | ❌ |
| Real-time sync | ✅ WebSocket / local server | ✅ | ✅ |
| Dev complexity for non-web-devs | Medium | High | Medium |
| Multi-device (3 judges) | ✅ All on same Wi-Fi | ✅ | Limited |
| Offline fallback | ✅ PWA caching | ✅ | ✅ |

**Architecture**: Run a lightweight **Node.js server** on one laptop/device (the "host"). All 3 judges connect via the same Wi-Fi LAN using their phones or tablets in a browser. No internet required. Real-time sync via **WebSocket**.

**Tech Stack (MVP)**
- **Backend**: Node.js + Express + Socket.io (WebSocket)
- **Frontend**: Plain HTML/CSS/JavaScript (no framework — keeps it accessible for non-web devs)
- **Data**: In-memory during match; **append-only JSON log file** written to disk for audit/correction
- **Hosting**: Local machine on LAN (e.g., `http://192.168.x.x:3000`)
- **Target devices**: Smartphones (primary), tablets (optional) — UI must be mobile-first, touch-friendly
- **QR code**: `qrcode` npm package — server auto-detects LAN IP at startup and generates a QR code rendered on the Operator screen; judges scan once to open their role-selection page

### Configuration File
A `config.json` at project root controls all tunable parameters:
```json
{
  "confirmationWindowMs": 2000,
  "complaintTimeoutMode": "timed",
  "complaintTimeoutMs": 30000,
  "roundDurationMs": 120000,
  "restDurationMs": 60000
}
```
All timing values are editable before a match starts without touching any code.

---

## 4. Core Mechanic — Dual Confirmation Window

One of the most critical rules in this app:

> When **Juri 1** OR **Juri 2** registers a point (specifying both the corner and the technique), the **other judge** must confirm the **same corner AND the same technique** within the **confirmation window** (default: 2 seconds, configurable). If the second judge does not confirm — or confirms a different corner or technique — the point is **invalidated/skipped**.

### Logic Flow
```
Juri 1 selects corner = "Merah", technique = "Kick (2pt)"
  → Server records a pending entry: { corner: "merah", technique: "kick", points: 2, initiator: "juri_1" }
  → Server starts a configurable countdown (default 2s)
  → Juri 2 must select corner = "Merah" AND technique = "Kick" within the window
    → If YES (corner + technique both match) → point is valid, scoreboard updates, event written to log
    → If mismatch (wrong corner or wrong technique) → point is cancelled
    → If NO confirmation within window / timeout → point is cancelled, alert shown, cancellation written to log
```

This mirrors how real Juri must independently agree on both the technique and which athlete scored.

> **Key change from previous design**: the corner is no longer fixed to a judge's role. Both judges observe both athletes and independently specify who they believe scored. The confirmation now requires matching on **corner + technique**, not just technique.

> **Config key**: `confirmationWindowMs` in `config.json`

---

## 5. Complaint / Protes Mechanic

### 5.1 Rule Clarification & Research Finding

> **Important distinction**: Official PERSILAT rules treat protests as a **post-match written procedure** (submitted within 15–20 minutes after the match, with a fee). There is no official in-match real-time protest dialog in the PERSILAT rulebook.
>
> However, the complaint dialog described below is a **widely used local/IPSI convention** in Indonesian regional competitions (Popda, Porprov, Kejurda, etc.), where a match official or corner coach can raise an immediate in-match objection that the judging panel resolves on the spot. We build to **match how this app is actually used in the field**, not the international written-protest procedure.

### 5.2 Who Can Raise a Complaint

A corner coach raises a complaint **verbally** to the match official. The **Operator** (on the operator device) then triggers it digitally in the app. Judges and corner judges cannot initiate complaints directly.

### 5.3 Complaint Selection by Operator

When triggering a complaint, the Operator is presented with:

1. **Predefined complaint list** — common reasons the Operator can tap to select:
   - Nilai tidak tercatat (Score not recorded)
   - Nilai salah dicatat (Incorrect score recorded)
   - Pelanggaran tidak dicatat (Foul not recorded)
   - Pelanggaran salah dicatat (Incorrect foul recorded)
   - Teknik tidak sah dinilai (Invalid technique was scored)
   - Nilai ganda / duplikat (Duplicate point entry)

2. **Free-form text input** — if no predefined reason fits, the Operator types a short description

3. **Notice-only fallback** — if the Operator just needs to flag an event without specifying a reason, they can send a plain "Protes diterima" notice that appears on all screens (no vote required, just acknowledgment)

### 5.4 Complaint Flow

```
Operator selects complaint reason (predefined / free-form / notice-only)
  → Round timer is paused (if not already)
  → A modal appears on ALL three judges' screens simultaneously, showing:
      - Complaint reason/description
      - Three vote buttons: 🔴 Merah | 🔵 Biru | ❌ Tidak Sah
  → Each judge independently taps their vote
  → Server collects votes; majority wins (2 of 3)
  → Result broadcast to all screens → scoreboard updates → event written to log
  → Operator resumes the round timer manually
```

**Notice-only path** (no vote):
```
Operator sends notice-only complaint
  → All screens show a banner: "Protes diterima — menunggu keputusan wasit"
  → No vote, no score change
  → Operator dismisses the notice when ready
  → Event written to log as "notice"
```

### 5.5 Outcome Logic

| Votes | Result |
|---|---|
| 2+ vote Merah | Complaint approved → score corrected in favor of Sudut Merah |
| 2+ vote Biru | Complaint approved → score corrected in favor of Sudut Biru |
| 2+ vote Tidak Sah | Complaint rejected → score unchanged |

### 5.6 Complaint Timeout Modes

> **Config key**: `complaintTimeoutMode` and `complaintTimeoutMs` in `config.json`

| Mode | Behavior |
|---|---|
| `"timed"` | Dialog closes automatically after `complaintTimeoutMs` (e.g., 30 000 ms). Majority of votes received at that point is used. If no votes or a tie, defaults to "Tidak Sah". |
| `"manual"` | Dialog stays open indefinitely until the Operator explicitly closes/resolves it (e.g., when the head referee signals a decision). No automatic timeout. |

### 5.7 UI Behavior During Complaint

- All scoring and foul inputs are **locked** on all judge screens while a complaint dialog is active
- A visible countdown (if in `timed` mode) is shown inside the dialog
- Complaint can only be triggered while the round timer is **paused or stopped**
- Each complaint event is written to the match log with: timestamp, reason, individual votes, outcome

### 5.8 Bahasa Indonesia UI Labels for Complaint Dialog

| Internal key | Label shown on screen |
|---|---|
| `complaint_title` | Protes / Keberatan |
| `complaint_prompt` | Tentukan hasil keberatan: |
| `complaint_reason_label` | Alasan: |
| `complaint_freeform_placeholder` | Tulis alasan protes... |
| `complaint_notice_only` | Protes diterima — menunggu keputusan wasit |
| `vote_merah` | Merah |
| `vote_biru` | Biru |
| `vote_invalid` | Tidak Sah |
| `complaint_resolved` | Keberatan selesai |
| `complaint_timeout` | Waktu habis — keberatan ditolak |
| `complaint_dismissed` | Protes ditutup oleh operator |

---

## 6. Match Audit Log

Logging is a core feature, not optional. Every meaningful event in a match is written to an append-only log file on the server. This allows post-match review and manual correction if something goes wrong mid-match (e.g., wrong button tapped, connectivity blip, contested entry).

### 6.1 Log Format

Each log entry is a single JSON object, one per line (newline-delimited JSON / NDJSON). Example entries:

```json
{"seq":1,"ts":"2026-05-03T08:01:00.123Z","type":"match_start","round":1}
{"seq":2,"ts":"2026-05-03T08:01:05.210Z","type":"score_pending","corner":"merah","technique":"kick","points":2,"initiator":"juri_1"}
{"seq":3,"ts":"2026-05-03T08:01:05.980Z","type":"score_confirmed","corner":"merah","technique":"kick","points":2,"confirmedBy":"juri_2","elapsed_ms":770}
{"seq":4,"ts":"2026-05-03T08:01:22.000Z","type":"score_pending","corner":"biru","technique":"hand","points":1,"initiator":"juri_2"}
{"seq":5,"ts":"2026-05-03T08:01:24.100Z","type":"score_invalidated","corner":"biru","technique":"hand","points":1,"reason":"timeout"}
{"seq":6,"ts":"2026-05-03T08:01:40.000Z","type":"foul","corner":"merah","level":"peringatan_1","deduction":-1,"by":"juri_pelanggaran"}
{"seq":7,"ts":"2026-05-03T08:02:00.000Z","type":"complaint","reason":"Nilai tidak tercatat","votes":{"juri_merah":"merah","juri_biru":"merah","juri_pelanggaran":"tidak_sah"},"outcome":"merah","score_delta":{"merah":+1}}
{"seq":8,"ts":"2026-05-03T08:03:00.000Z","type":"round_end","round":1,"score":{"merah":5,"biru":3}}
{"seq":9,"ts":"2026-05-03T08:07:00.000Z","type":"pencak_point","round":1,"corner":"biru","by":"juri_merah"}
{"seq":10,"ts":"2026-05-03T08:10:00.000Z","type":"match_end","winner":"merah","final_score":{"merah":12,"biru":9}}
```

### 6.2 Log Event Types

| Event type | When it fires |
|---|---|
| `match_start` | Operator starts the match |
| `round_start` / `round_end` | Round timer starts / ends |
| `score_pending` | A corner judge submits a technique |
| `score_confirmed` | Both judges agreed within the window |
| `score_invalidated` | Timeout or mismatch — point rejected |
| `foul` | Juri Pelanggaran records a violation |
| `pencak_point` | End-of-round Pencak Point awarded |
| `complaint` | A protes is raised, voted on, and resolved |
| `complaint_notice` | Notice-only complaint (no vote) |
| `timer_pause` / `timer_resume` | Operator pauses/resumes the round |
| `match_end` | Final result declared |

### 6.3 Log File Location & Naming

```
sudut-silat/
└── logs/
    └── match_20260503_0801.ndjson   # One file per match, named by date+time
```

### 6.4 Post-Match Use

- The log file can be opened in any text editor or spreadsheet
- Future enhancement: an `/admin` page on the server to browse and replay the log
- Manual correction: if an entry was wrong, a corrective entry is appended (never delete — append-only). Example:
  ```json
  {"seq":11,"ts":"...","type":"correction","corrects_seq":3,"reason":"Juri salah tekan","new_score_delta":{"merah":-2}}
  ```

---

## 7. Language — Bahasa Indonesia Rendering

All **user-facing text** (labels, buttons, dialogs, scoreboard, notifications) is rendered in **Bahasa Indonesia**. All **code, variable names, comments, documentation, and planning** remain in **English**.

### Key UI String Map (English → Bahasa Indonesia)

| English (internal) | Bahasa Indonesia (rendered) |
|---|---|
| Red Corner | Sudut Merah |
| Blue Corner | Sudut Biru |
| Judge 1 | Juri 1 |
| Judge 2 | Juri 2 |
| Foul Judge | Juri Pelanggaran |
| Select corner | Pilih sudut |
| Scoreboard | Papan Skor |
| Round | Babak |
| Match | Pertandingan |
| Start | Mulai |
| Stop | Berhenti |
| Pause | Jeda |
| Reset | Ulang |
| Hand Strike (1pt) | Pukulan (1) |
| Evade + Hand (2pt) | Elak + Pukulan (2) |
| Kick (2pt) | Tendangan (2) |
| Evade + Kick (3pt) | Elak + Tendangan (3) |
| Takedown (3pt) | Jatuhan (3) |
| Evade + Takedown (4pt) | Elak + Jatuhan (4) |
| Pending confirmation... | Menunggu konfirmasi... |
| Point confirmed! | Nilai sah! |
| Point invalidated | Nilai tidak sah |
| Warning | Teguran |
| Warning I | Peringatan I |
| Warning II | Peringatan II |
| Penalty I | Hukuman I |
| Penalty II / DQ | Hukuman II / Diskualifikasi |
| Pencak Point | Nilai Pencak |
| Award Pencak Point | Beri Nilai Pencak |
| Winner | Pemenang |
| Draw | Seri |
| Match Log | Log Pertandingan |
| Correction | Koreksi |

> This string map will eventually live in a dedicated `i18n/id.js` file in the codebase.

---

## 7. MVP Feature List

### 7.1 Role Assignment
- On launch, each device selects a role: `Juri 1`, `Juri 2`, or `Juri Pelanggaran`
- A 4th view exists: **Scoreboard** (display-only, for projector/audience screen)
- A 5th view: **Operator / Match Control** (starts/pauses round timer, raises complaints)

### 7.2 Scoring Interface (Juri 1 & Juri 2)

Both judges use an identical, **3-step scoring flow** per entry:

**Step 1 — Pilih sudut (which corner scored)**
- Two large buttons: Merah | Biru
- Must be selected before proceeding

**Step 2 — Elak / tangkisan? (+1 bonus)**
- Two buttons: Ya, ada elak (+1 bonus) | Tidak, langsung (no bonus)
- Must be selected before proceeding

**Step 3 — Pilih jenis serangan**
- Three buttons unlocked after steps 1 and 2: Pukulan | Tendangan | Jatuhan
- Displayed total points update in real time based on step 2 selection

After step 3 is tapped:
- Pending confirmation banner appears with countdown
- The other judge must submit the same corner + same technique within the window
- Visual + audio feedback on confirmation or invalidation
- UI resets to step 1 for the next entry

**Pencak Point button** — available only at end of round (enabled by Operator); judge selects Merah or Biru; one award per judge per round

### 7.3 Foul Interface (Juri Pelanggaran)
- Select **which athlete** (Merah / Biru) received the violation
- Select **violation level**: Teguran / Peringatan I / Peringatan II / Hukuman I / Hukuman II
- Confirm button — deduction applied immediately (no dual-confirmation needed)
- Log of all recorded fouls in the session

### 7.4 Complaint Dialog (All Judge Screens)
- Triggered by the Operator
- Locks all scoring inputs on all devices
- Shows a modal with: complaint description + three vote buttons (Merah / Biru / Tidak Sah)
- Countdown timer visible on the dialog (e.g., 30 seconds)
- Result broadcast to all screens after majority is reached or timeout

### 7.5 Scoreboard (Shared View)
- Live score: Merah | Round | Biru
- Round indicator (R1 / R2 / R3)
- Match timer (countdown per round)
- Foul count per athlete
- Visual flash when a point is scored or deducted

### 7.6 Match Control (Operator View)
- Start / Pause / Stop Round timer
- End Round → triggers Pencak Point phase → auto-advance to rest
- End Match → show final result
- Reset / New Match
- Trigger complaint / protes (with reason picker or free-form)
- Enable/disable Pencak Point phase per round

### 7.7 Pencak Point Phase
- Triggered by Operator at the **end of each round**, before rest countdown begins
- Each of the 3 judges independently awards 0 or 1 Pencak Point to **either** athlete (or neither)
- No dual-confirmation needed — it's a subjective, independent judgment
- Each award written to the log; total added to round score
- Phase closes only when **all 3 judges have submitted** — the phase does not auto-close or time out; every judge must cast their Pencak Point before the round ends

---

## 8. Development Phases

### Phase 0 — Setup (Week 1)
- [ ] Install Node.js on host machine
- [ ] Scaffold project: `server.js`, `public/` folder
- [ ] Get all 3 devices connected on same Wi-Fi
- [ ] Basic "Hello from server" test via browser on all devices

### Phase 1 — MVP Core (Week 2–3)
- [ ] `config.json` with all configurable parameters
- [ ] i18n string file (`public/i18n/id.js`) for all UI text
- [ ] Role selection screen (Bahasa Indonesia labels)
- [ ] WebSocket connection & room management (Socket.io)
- [ ] Score submission from Juri 1 / Juri 2 (corner selected per entry, not fixed per role)
- [ ] Dual-confirmation window (configurable timer)
- [ ] Point validation / invalidation broadcast
- [ ] Append-only match log written to `logs/` on every event
- [ ] Foul submission from Juri Pelanggaran
- [ ] Live scoreboard view

### Phase 2 — Match Flow (Week 3–4)
- [ ] Round timer (configurable duration, configurable rest)
- [ ] Operator view: start / pause / stop / end round / end match
- [ ] Round-by-round score tracking
- [ ] Pencak Point phase at end of each round
- [ ] Winner declaration logic
- [ ] Complaint/protes dialog (predefined list + free-form + notice-only)
- [ ] Input lock during active complaint
- [ ] Configurable complaint timeout (timed or manual)

### Phase 3 — Polish & Testing (Week 5)
- [ ] Mobile-first UI (large tap targets, min 48px, readable fonts at arm's length)
- [ ] Audio feedback (confirm beep, reject buzz, timer warning)
- [ ] Reconnect handling — judge rejoins mid-match without data loss (log is source of truth)
- [ ] QR code displayed on Operator screen — judges scan to connect instantly, no IP typing needed

### Phase 4 — Post-MVP
- [ ] Out-of-arena step counter per athlete (3 steps = lose round)
- [ ] Knockdown counter (3 knockdowns = RSC)
- [ ] `/admin` log viewer in browser — replay match event by event
- [ ] **In-app correction UI** — Operator selects a past log entry and appends a corrective record; no manual file editing required
- [ ] Multi-match / bracket support

---

## 9. Decisions Log

All open questions from planning are resolved. Recorded here for traceability.

| Question | Decision |
|---|---|
| Judge devices | Smartphones (primary), tablets (optional) — mobile-first UI |
| Confirmation window duration | Configurable via `config.json` (default 2 000 ms) |
| Pencak Point in MVP | ✅ Yes, mandatory for MVP |
| Match audit log | ✅ Mandatory — append-only NDJSON per match in `logs/` |
| Who controls match timer | Dedicated **Operator** device/role |
| Who triggers complaint | **Operator only** — corner coach raises verbally, Operator invokes in app |
| Complaint reason format | Predefined list first → free-form fallback → notice-only as last resort |
| Complaint timeout | Configurable: `"timed"` (default 30 s) or `"manual"` (Operator closes) |
| Judge connection method | **QR code** on Operator screen — judges scan to connect; no manual IP entry |
| Pencak Point phase closing | **All 3 judges must submit** before phase closes — no force-close, no timeout |
| Correction workflow | **In-app UI** (Phase 4) — Operator selects a past log entry, appends corrective record; log stays append-only |
| Corner assignment per judge | **Not fixed at role selection** — both Juri 1 and Juri 2 select which corner scored on each individual entry; dual-confirmation requires matching corner + technique |

> ✅ All planning decisions resolved. No remaining open items.

---

## 10. File Structure (Planned)

```
sudut-silat/
├── server.js                    # Node.js + Express + Socket.io backend
├── config.json                  # All configurable parameters (timers, timeouts)
├── package.json
├── logs/                        # Append-only match logs (auto-created)
│   └── match_YYYYMMDD_HHMM.ndjson
├── public/
│   ├── index.html               # Role selection landing page
│   ├── juri-1.html              # Judge 1 interface (scores either corner)
│   ├── juri-2.html              # Judge 2 interface (scores either corner)
│   ├── juri-pelanggaran.html    # Foul judge interface
│   ├── scoreboard.html          # Display-only scoreboard
│   ├── operator.html            # Match control + complaint trigger + QR code display
│   ├── style.css                # Mobile-first shared styles
│   ├── client.js                # Shared Socket.io client logic
│   └── i18n/
│       └── id.js                # All Bahasa Indonesia UI strings
├── PLAN.md                      # This file
└── README.md                    # Setup & run instructions
```

---

## 11. Sources

- [Tanding – PERSILAT Ruleset (USSSA)](https://usasportsilat.org/compete/point-sparring-persilat-ruleset/)
- [Pencak Silat 2024 Competition Rules & Regulations](https://6212555.fs1.hubspotusercontent-na1.net/hubfs/6212555/Pesta%20Sukan%202024/Rules%20and%20Regulations/Silat/PESTA%20SUKAN%202024%20RULES%20AND%20REGULATIONS_17April.pdf)
- [Scoring in Silat – ActiveSG](https://www.activesgcircle.gov.sg/learn/silat/scoring-in-silat)
- [Macam-macam Pelanggaran dalam Pencak Silat – Kompas](https://www.kompas.com/sports/read/2021/12/20/21200098/macam-macam-pelanggaran-dalam-pencak-silat)
- [Peraturan Pertandingan Pencak Silat – Kompas](https://www.kompas.com/sports/read/2023/01/27/16200088/peraturan-dalam-pertandingan-pencak-silat?page=all)
- [PERSILAT Competition Regulation – Indian Pencak Silat](https://indianpencaksilat.org/rules_and_regulations/Wasit-Jury.pdf)
- [Pencak Silat Competition Rules Ver 7 (Oct 2023) – Scribd](https://www.scribd.com/document/699349333/1-Pencak-Silat-Competition-Rules-and-Regulations-Ver-7-09-10-2023-6-7-1)
- [Macam-macam Pelanggaran Berat – Mingseli](https://www.mingseli.id/2019/12/macam-macam-pelanggaran-berat.html)
