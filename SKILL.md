---
name: new-project-scheduler
description: >
  New Project Automation Scheduler — connects your calendar, active projects, and NPAO task queues to build a real, time-aware daily and weekly work plan. Reads your connected calendar to find available slots, pulls NPAO task queues from all active projects, and schedules each task into your actual day — modularized to fit real time blocks, sequenced by N→A→P→O priority, and tracked against project milestones. ALWAYS use when anyone says: "plan my day", "schedule my tasks", "what should I work on today", "fit this into my week", "when can I build X", "project timeline", "milestone planning", "when will this be done", "block time for this project", "organize my week around my projects", "daily build plan", "what's the NPAO priority for today", "fit the JTBD tasks into my calendar", "I have X hours — what do I build", "sprint plan", "work schedule", or any request to connect active project tasks to real calendar time. Also triggers when a project is approved for build and needs to be ...
---

# New Project Automation — Scheduler Skill

This skill bridges your **NPAO task queues** and **active project build plans** with your **real calendar** — so every task gets a time slot, every milestone has a date, and every workday has a clear, prioritized plan built around what actually matters.

It doesn't suggest tasks. It schedules them — into the specific open windows on your calendar, in the right order, for the right project.

---

## Framework References

All scheduling logic in this skill is governed by the **ROSTR Framework**. The canonical source:

> **ROSTR Research Paper:** https://rostr-paper.vercel.app
> *{{USER_NAME}} — GTM AI & Automation Manager, Enterprise Platform*

When any framework term or behavior is unclear, fetch the live paper. Do not guess at definitions.

| Module | Role in This Skill | Reference |
|---|---|---|
| **NPAO** — Priority Framework | Determines scheduling order (N→A→P→O) and time block assignment per class | https://rostr-paper.vercel.app/#s6 |
| **4Ds Lifecycle** | Phase awareness — D3 Deploy tasks get scheduling priority over D1 Design | https://rostr-paper.vercel.app/#s6-7 |
| **PAL** | Used to compile task estimates and classify any ambiguous task inputs | https://rostr-paper.vercel.app/#s4 |
| **JTBD Builder** | Source of all task queues, build prompts, and done-when criteria | `JTBD_BUILDER_SKILL.md` |
| **Intake Skill** | Source of project milestones, timelines, and constraints | `PROJECT_INTAKE_SKILL.md` |

**NPAO Scheduling Rules (from paper — non-negotiable):**
- **N (Necessity):** Schedule into earliest available slot. Cannot be pushed past today without a BLOCKED flag.
- **A (Anxiety):** Schedule before **any** P-class task. Unresolved anxiety degrades P-class execution quality.
- **P (Priority):** Requires Deep Work block (90+ min). Never schedule into fragmented time.
- **O (Opportunity):** Fills remaining capacity only. Never displaces N, A, or P. Waiting is correct behavior.
- **Anxiety escalation:** Open A-class task > 3 days → escalate to N-class automatically.

Full NPAO specification: https://rostr-paper.vercel.app/#s6

---

## What This Skill Does

1. **Reads your calendar** — finds open time blocks for the day, week, or sprint
2. **Reads all active projects** — pulls NPAO task queues from every project in the directory
3. **Prioritizes across projects** — applies N→A→P→O across all active queues simultaneously
4. **Schedules tasks into time blocks** — assigns each task to a specific slot based on duration estimate and calendar availability
5. **Tracks against milestones** — flags when a project is behind, on track, or at risk
6. **Outputs a daily/weekly plan** — modular, calendar-ready, formatted for Asana or direct use

---

## Calendar Integration

### What the system reads:

| Data Source | What It Pulls | Used For |
|---|---|---|
| Calendar (Outlook / Google) | Events, meetings, blocks, OOO | Finding available work windows |
| Asana (connected projects) | Tasks, due dates, assignees, dependencies | Loading NPAO queues |
| Project Directory | `JTBD_BUILD_PLAN.md` per project | Loading build tasks + estimates |
| `PROJECT_INDEX.md` | All active projects + phases | Priority ranking across projects |

### Available time block detection:

The system scans the calendar and classifies each slot:

| Block Type | Definition | Schedulable? |
|---|---|---|
| **Deep Work** | 90+ min uninterrupted | N-class and P-class tasks (complex builds) |
| **Focus** | 45–90 min | P-class tasks (standard builds, research) |
| **Quick Hit** | 15–45 min | A-class tasks (decisions, unblocks, reviews) |
| **Transition** | <15 min | O-class tasks (async sends, updates, logging) |
| **Meeting** | Scheduled event | Not schedulable — shown as context |
| **Buffer** | Intentional gap | Reserved — not filled unless user requests |

---

## NPAO Scheduling Rules

The NPAO class of each task determines **when** it gets scheduled, not just whether it gets scheduled.

```
SCHEDULING PRIORITY ORDER:
════════════════════════════════════════
N — NECESSITY  →  Schedule FIRST
  Assigned to the earliest available Deep Work or Focus block.
  Nothing else is scheduled until all N-class tasks for active
  projects have a slot or a confirmed completion date.
  If no slot exists today → flag as BLOCKED and surface to user.

A — ANXIETY  →  Schedule SECOND
  Assigned before any P-class tasks. Prefer Focus or Quick Hit blocks.
  Unresolved anxiety degrades execution quality on P-class work.
  Anxiety tasks that have been open >3 days get escalated to N-class.

P — PRIORITY  →  Schedule THIRD
  Core mission work. Requires Deep Work blocks.
  Never scheduled into fragmented time (back-to-back meetings).
  If no Deep Work block available today → schedule for next available day.
  Flag if P-class tasks are being pushed >2 consecutive days.

O — OPPORTUNITY  →  Schedule LAST
  Fills remaining capacity after N, A, P are placed.
  If calendar is full → O-class tasks are not scheduled (they wait).
  Never bumps N, A, or P to fit.
════════════════════════════════════════
```

---

## Task Duration Estimation

Before scheduling, estimate the duration of each task. If the JTBD Build Plan includes estimates, use those. Otherwise apply defaults:

| Task Type | Default Estimate |
|---|---|
| API setup / authentication | 30 min |
| Schema / data model design | 45 min |
| n8n workflow build (simple) | 60 min |
| n8n workflow build (complex) | 90–120 min |
| Claude skill / agent build | 60–90 min |
| HubSpot config / property setup | 30 min |
| Clay table build | 45–60 min |
| Research / documentation | 30–45 min |
| PRD / spec writing | 60 min |
| Testing and QA | 30–60 min |
| Stakeholder review / async update | 15–20 min |
| Deployment / publish | 20–30 min |

Add **20% buffer** to any task that has dependencies or unconfirmed access.

---

## Cross-Project Prioritization

When multiple active projects have tasks ready to schedule, apply this ranking to decide which project's tasks go first:

| Priority Factor | Weight | Signal |
|---|---|---|
| **Hard deadline** | Highest | Due date within 3 days |
| **NPAO class** | High | N-class from any project beats P-class from another |
| **Milestone risk** | High | Project flagged as behind or at risk |
| **4Ds phase** | Medium | D3 (Deploy) tasks take priority over D1 (Design) |
| **Leadership visibility** | Medium | Projects where leadership has asked for status |
| **CoE gap** | Low | Undocumented projects bumped ahead of documented ones |
| **O-class only** | Lowest | Projects with only O-class tasks remaining |

---

## Daily Plan Output

The scheduler produces a structured daily plan. Each task is assigned to a specific time slot, linked to its project, labeled with NPAO class and 4Ds phase.

```
══════════════════════════════════════════
📅 DAILY BUILD PLAN — [Day, Date]
Generated: [time] | Working hours: [X:XX – X:XX] | Available: [X hrs]
══════════════════════════════════════════

MEETINGS TODAY (not schedulable):
  [Time] [Meeting name] ([duration])
  [Time] [Meeting name] ([duration])

──────────────────────────────────────────
SCHEDULED TASKS
──────────────────────────────────────────

[TIME] – [TIME]  ▓ DEEP WORK ([X min])
  [N] [Project Name] — [Task ID]: [Task Name]
  4Ds: [phase] | Est: [X min] | Done when: [criterion]
  Build prompt: [first sentence — full prompt in JTBD_BUILD_PLAN.md]

[TIME] – [TIME]  ▓ FOCUS ([X min])
  [A] [Project Name] — [Task ID]: [Task Name]
  4Ds: [phase] | Est: [X min] | Resolves: [what friction]

[TIME] – [TIME]  📅 MEETING
  [Meeting name]

[TIME] – [TIME]  ▓ FOCUS ([X min])
  [P] [Project Name] — [Task ID]: [Task Name]
  4Ds: [phase] | Est: [X min] | Feeds: [next task]

[TIME] – [TIME]  ▒ QUICK HIT ([X min])
  [A] [Project Name] — [Task ID]: [Task Name]
  Est: [X min] | Quick: confirm [thing] so [task] can proceed

[TIME] – [TIME]  ▒ QUICK HIT ([X min])
  [O] [Project Name] — [Task ID]: [Task Name]
  Est: [X min] | Optional — skip if running behind

──────────────────────────────────────────
PUSHED TO TOMORROW (no slot available today):
  [N] [Project] — [Task] — ⚠ FLAGGED: N-class with no slot
  [P] [Project] — [Task] — needs Deep Work block

──────────────────────────────────────────
DAILY SUMMARY
  Scheduled: [X] tasks | [X hrs] of work
  N-class covered: [X/X] | A-class: [X/X] | P-class: [X/X] | O-class: [X/X]
  Projects touched today: [list]
══════════════════════════════════════════
```

---

## Weekly Plan Output

The weekly view shows the full sprint — tasks distributed across the week with milestone markers.

```
══════════════════════════════════════════
📆 WEEKLY BUILD PLAN — Week of [Date]
Projects active: [X] | Total tasks queued: [X] | Capacity: ~[X hrs]
══════════════════════════════════════════

MONDAY [Date] — [X hrs available]
  [N] [Project A] — N1: [task]  ([X min])
  [A] [Project B] — A2: [task]  ([X min])
  [P] [Project A] — P1: [task]  ([X min])

TUESDAY [Date] — [X hrs available]
  [P] [Project A] — P2: [task]  ([X min])
  [P] [Project B] — P1: [task]  ([X min])
  [A] [Project C] — A1: [task]  ([X min])

WEDNESDAY [Date] — [X hrs available]
  ⚠ MILESTONE: [Project A] — [Milestone name] due
  [P] [Project A] — P3: [task]  ([X min])
  [P] [Project A] — P4: [task]  ([X min])

THURSDAY [Date] — [X hrs available]
  [P] [Project B] — P2: [task]  ([X min])
  [O] [Project A] — O1: [task]  ([X min]) — if P-class complete

FRIDAY [Date] — [X hrs available]
  [P] [Project B] — P3: [task]  ([X min])
  ✓ REVIEW: weekly milestone check + PROJECT_INDEX.md update
  [O] [Project B] — O1: [task]  ([X min]) — if time permits

──────────────────────────────────────────
MILESTONE TRACKER
  [Project A] → [Milestone] — ON TRACK ✓ (due [date], [X] tasks left)
  [Project B] → [Milestone] — AT RISK ⚠ (due [date], [X] tasks left, [X hrs] available)
  [Project C] → [Milestone] — BLOCKED ❌ (N-class unresolved: [task])

──────────────────────────────────────────
WEEKLY SUMMARY
  Tasks planned: [X] | Capacity used: [X%]
  N complete: [X/X] | A complete: [X/X] | P complete: [X/X]
  Projects advancing: [list]
  Projects at risk: [list]
══════════════════════════════════════════
```

---

## Milestone Tracking

Every project in the directory has milestone definitions from its PRD. The scheduler tracks each milestone against the current task queue and calendar capacity.

**Milestone status rules:**

| Status | Condition |
|---|---|
| **ON TRACK ✓** | Remaining tasks fit within available capacity before due date |
| **AT RISK ⚠** | Remaining tasks require more capacity than available at current pace |
| **BLOCKED ❌** | One or more N-class tasks have no scheduled slot or unresolved dependency |
| **BEHIND 🔴** | Milestone date has passed and tasks remain incomplete |
| **COMPLETE ✅** | All P-class tasks done; O-class optional |

**AT RISK escalation:**
When a milestone is flagged AT RISK, the system:
1. Identifies the specific tasks causing the gap
2. Calculates how much additional capacity is needed
3. Suggests: reschedule lower-priority tasks, reduce scope to O-class, adjust milestone date, or add a focus day

**Milestone check format:**
```
⚠ MILESTONE AT RISK — [Project Name]
Target: [Milestone name] by [date]
Gap: [X tasks] need [X hrs], only [X hrs] available before deadline

Options:
  A. Move [O1, O2, O3] to post-milestone → saves [X hrs]
  B. Add 1 deep work day on [date] → covers gap
  C. Adjust milestone to [suggested date]
  D. Reduce scope: cut [specific task] — impact: [what's lost]

What would you like to do?
```

---

## Daily Check-In Mode

When the user says "what's on my plate today" or "what should I work on", run a fast daily check-in:

```
DAILY CHECK-IN — [Day, Date]
──────────────────────────────────────
CARRY-OVER FROM YESTERDAY:
  [Any unfinished tasks from prior day's plan]

TODAY'S NPAO PRIORITY:
  🔴 N-class due: [task — project — est time]
  🟡 A-class clearing: [task — project — est time]
  🟢 P-class build: [task — project — est time]

YOUR FIRST TASK RIGHT NOW:
  [Task ID] — [Task Name]
  Project: [name] | Phase: [4Ds] | Est: [X min]
  Done when: [criterion]
  Build prompt: [first sentence]

[START THIS TASK] [SHOW FULL DAY PLAN] [RESCHEDULE]
```

---

## End-of-Day Wrap

At end of day (or on request), run a quick wrap to update project status and prep tomorrow:

```
END OF DAY — [Date]
──────────────────────────────────────
COMPLETED TODAY:
  ✓ [Task] — [Project] ([X min actual vs est])
  ✓ [Task] — [Project]

INCOMPLETE (carrying over):
  ○ [Task] — [Project] — why: [reason]

PROJECT STATUS UPDATES:
  [Project A]: [phase] → [updated status]
  [Project B]: [phase] → P2 complete, P3 ready to schedule

TOMORROW'S FIRST TASK:
  [Task ID] — [Task Name] ([Project])
  Est: [X min] | Schedule: [time slot]

PROJECT_INDEX.md: [updated ✓]
──────────────────────────────────────
```

---

## Operating Modes

| Mode | Trigger | Output |
|---|---|---|
| **Daily Plan** | "Plan my day" / morning | Full day schedule with time blocks |
| **Weekly Plan** | "Plan my week" / Monday | Full week with milestones |
| **Sprint Plan** | "Plan this sprint" / 2-week window | Sprint board with NPAO queue |
| **Quick Priority** | "What do I work on now?" | Single next task with context |
| **Milestone Check** | "Are my projects on track?" | Milestone status for all active projects |
| **Capacity Check** | "Can I take on a new project?" | Available capacity vs. current load |
| **Daily Check-In** | "What's on my plate today?" | Fast NPAO priority summary |
| **End of Day** | "Wrap up / EOD" | Completion log + tomorrow prep |
| **Reschedule** | Task blocked / ran long | Reflow remaining tasks around the gap |

---

## Reflow Logic

When a task runs over estimate or a blocker is hit mid-day, the scheduler reflows:

1. Identify how much time was lost
2. Identify which remaining tasks can be compressed or moved
3. Apply NPAO rules: N-class stays today, P-class moves to next Deep Work block, O-class drops
4. Present updated plan: "Here's the adjusted day — [X] tasks rescheduled"

```
REFLOW — [Time]
[Task N1] ran [X min] over estimate.
Adjusted plan:
  [P2] moved to tomorrow [time slot]
  [O1] dropped (no capacity)
  [A2] kept — Quick Hit at [time]

New EOD target: [tasks remaining today]
```

---

## NPAO Quick Reference (Scheduling Context)

```
N — NECESSITY: Schedule into the EARLIEST available slot.
  Cannot be pushed past today without flagging as BLOCKED.

A — ANXIETY: Schedule before ANY P-class task.
  If anxiety is unresolved, P-class execution quality degrades.
  Open >3 days → escalate to N-class.

P — PRIORITY: Requires Deep Work block (90+ min uninterrupted).
  Never schedule into fragmented time.
  Pushed >2 consecutive days → flag as milestone risk.

O — OPPORTUNITY: Fills remaining capacity only.
  Never displaces N, A, or P.
  If no slot → it waits. This is correct behavior.
```

---

## Enterprise Platform GTM Auto-Context

- Patrick's work hours: assumed standard Chicago business hours unless calendar says otherwise
- Projects are GTM automations — most P-class tasks are builds (Clay, n8n, HubSpot, Amplemarket)
- Deep Work blocks are precious — protect them for actual builds, not admin
- Meetings are assumed non-movable unless explicitly flagged
- O-class tasks are often dashboard improvements or Slack notifications — low urgency

---

## Output Quality Rules

Every plan must be:
- **Time-specific** — each task has a start time and end time, not just "morning"
- **NPAO-sequenced** — N before A before P before O, always
- **Milestone-aware** — at-risk projects surfaced, not buried
- **Realistic** — doesn't schedule 8 hours of Deep Work into a day with 4 hours of meetings
- **Actionable** — the first task of the day is always specific enough to start immediately

Never:
- Schedule P-class tasks before all N-class tasks for the same project have slots
- Fill O-class tasks into Deep Work blocks while P-class tasks are waiting
- Push N-class tasks without surfacing a BLOCKED flag to the user
- Produce a plan that exceeds realistic available capacity by more than 10%
