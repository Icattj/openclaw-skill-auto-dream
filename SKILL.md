---
name: auto-dream
description: Automatic memory consolidation inspired by Claude Code's autoDream/KAIROS. Runs as a background cron job every 24h — reviews recent sessions, consolidates memory, prunes stale entries, and updates MEMORY.md. Zero manual intervention needed. Use to enable always-on memory maintenance.
---

# Auto-Dream — Background Memory Consolidation

Inspired by Claude Code's leaked `autoDream` service and KAIROS always-on agent. Runs automatically as a cron job to keep your memory clean, current, and useful.

## What It Does

Every 24 hours (configurable), auto-dream:

1. **Orient** — Read MEMORY.md and recent daily files to understand current state
2. **Gather** — Scan last 24-48h of activity for new signals worth persisting
3. **Consolidate** — Update memory files: merge new info, fix contradictions, convert relative dates to absolute
4. **Prune** — Remove stale entries, trim MEMORY.md if over 200 lines, clean up outdated context

## How It Works

```
┌─────────────────────────────────────────────┐
│              AUTO-DREAM CYCLE               │
│                                             │
│  Cron fires (every 24h)                     │
│       ↓                                     │
│  Gate check:                                │
│   - 24h since last dream? ✓                 │
│   - New activity exists?  ✓                 │
│       ↓                                     │
│  Spawn isolated sub-agent                   │
│       ↓                                     │
│  Phase 1: Orient (read MEMORY.md)           │
│  Phase 2: Gather (scan daily files)         │
│  Phase 3: Consolidate (update memories)     │
│  Phase 4: Prune (trim, clean, index)        │
│       ↓                                     │
│  Log result to memory/dream-log.json        │
│  Update last-dream timestamp                │
│       ↓                                     │
│  Done (silent unless notable changes)       │
└─────────────────────────────────────────────┘
```

## Setup

### Install the cron job
The agent should create this cron on first use:

```
Schedule: every 24 hours (e.g., 03:00 UTC = 11:00 WITA)
Session: isolated (separate from main)
Payload: agentTurn with the dream prompt
Delivery: announce (notify Lucy only if significant changes)
```

### Manual trigger
Say: `"Dream now"` or `"Consolidate memory"`

## The Dream Prompt

This is what the sub-agent receives. It follows Claude Code's 4-phase structure adapted for OpenClaw:

```
# Dream: Memory Consolidation

You are performing a dream — a reflective pass over your memory files. 
Synthesize what you've learned recently into durable, well-organized memories.

Memory directory: ~/.openclaw/workspace/memory/
Main memory: ~/.openclaw/workspace/MEMORY.md

---

## Phase 1 — Orient

- Read MEMORY.md to understand the current long-term memory state
- List memory/ directory to see daily files and evaluations
- Skim the last 3 days of memory/YYYY-MM-DD.md files
- Check memory/lessons.json if it exists (L2 knowledge)
- Check memory/dream-log.json for last dream results

## Phase 2 — Gather Recent Signal

Look for new information worth persisting:

1. Recent daily files (last 48h) — new decisions, discoveries, events
2. Evaluation files (memory/evaluations/) — task quality patterns
3. Contradictions — facts in MEMORY.md that conflict with recent activity
4. Stale entries — references to completed/cancelled projects, outdated context

Focus on SIGNAL, not noise. Don't persist routine operations.

## Phase 3 — Consolidate

For each thing worth remembering:

- If it updates existing MEMORY.md content → edit in place
- If it's new long-term knowledge → add to appropriate section
- If it contradicts existing memory → fix the old entry
- Convert relative dates to absolute ("yesterday" → "2026-04-04")
- Merge duplicate entries

For lessons.json (if hierarchical-memory skill is active):
- Check if any L1 evaluations from last 7 days form patterns (3+ occurrences)
- Promote to L2 lessons if warranted

## Phase 4 — Prune and Index

MEMORY.md maintenance:
- Remove entries about completed/cancelled projects (unless lesson is universal)
- Remove outdated contact info, stale URLs, obsolete infrastructure details
- Keep under 200 lines — if over, prune least-referenced entries
- Ensure all sections are current and accurate

Daily file maintenance:
- Files older than 90 days → move to memory/archive/YYYY-MM/

## Phase 5 — Log

Write dream results to memory/dream-log.json:
{
  "timestamp": "ISO 8601",
  "changes": ["what was added/updated/removed"],
  "memory_md_lines_before": N,
  "memory_md_lines_after": N,
  "files_archived": N,
  "lessons_promoted": N,
  "notable": true/false
}

If notable=true (significant changes), summarize what changed.
If nothing changed, log "no changes needed" and exit quietly.

---

Return a brief summary of what you consolidated, updated, or pruned.
```

## Gate Logic (When to Dream)

Before running the dream, check:

1. **Time gate**: 24h since last dream (read `memory/dream-log.json` timestamp)
2. **Activity gate**: At least 1 daily file modified since last dream
3. **Lock**: No other dream currently running

If any gate fails, skip silently.

## Dream Log

`memory/dream-log.json`:
```json
{
  "lastDreamAt": "2026-04-04T03:00:00Z",
  "dreams": [
    {
      "timestamp": "2026-04-04T03:00:00Z",
      "changes": [
        "Updated Council agent table (added Metis)",
        "Removed stale DataFlow April 5 deadline reference",
        "Archived 3 daily files (>90 days old)",
        "Promoted 1 lesson: sub-agent timeout pattern"
      ],
      "memory_md_lines_before": 155,
      "memory_md_lines_after": 148,
      "files_archived": 3,
      "lessons_promoted": 1,
      "notable": true
    }
  ]
}
```

Keep only last 30 dream entries. Older ones get dropped.

## Integration with Hierarchical Memory

If the `hierarchical-memory` skill is installed, auto-dream also:
- Runs L1→L2 promotion (extract lessons from evaluations)
- Runs L2→L3 promotion if any lessons cross 0.8 confidence
- Reviews `memory/rules.md` for consistency — remove contradicted rules, add new ones from recent corrections
- Updates evaluation counter

This replaces the need for separate weekly/monthly cron jobs.

## Configuration

Default values (can be adjusted):
- `dreamIntervalHours`: 24
- `minActivityFiles`: 1 (minimum daily files since last dream)
- `dreamTimeUtc`: "03:00" (11:00 WITA — while Lucy sleeps)
- `maxMemoryMdLines`: 200
- `archiveAfterDays`: 90
- `maxDreamLogEntries`: 30
- `announceIfNotable`: true (send notification if significant changes)

## Safety

- Dream runs in **isolated session** — can't affect main conversation
- Read-heavy, write-light — mostly reads files, only writes memory files
- No network calls
- No code execution beyond file read/write
- If dream fails, logs error and retries next cycle
- MEMORY.md changes are additive/editorial, never destructive rewrites
