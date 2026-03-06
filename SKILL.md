---
name: claw-history
description: Build an auditable chronological history of the agent’s actions with explicit coverage boundaries and evidence labels. Use when the user asks for “from birth until now,” “everything you’ve done,” “full action log,” timeline/accountability reports, or similar history requests. If full lifetime coverage is not possible, return a clearly marked partial history with gaps and reasons.
---

# Claw History

Return the most complete **evidence-backed** timeline possible.

## Non-Negotiables

1. Never claim full lifetime history unless coverage supports it.
2. Never silently fall back to current-session-only history.
3. If visibility is limited, label output as **PARTIAL** and explain why.
4. Never invent timestamps, actions, or sources.
5. Redact secrets/tokens/private identifiers when quoting logs.

## Preflight (Must Run First)

Before building history, determine what is actually visible:

1. Session visibility scope (all vs tree-restricted vs current session only).
2. History API limits/caps (max page size, truncation flags, dropped messages).
3. Available files (`memory/`, `MEMORY.md`, optional command logs).
4. Inaccessible sources (permission denied, not found, disabled hooks).

If any limitation exists, carry it into final output under **Gaps / limits**.

## Data Sources (Use in Order)

1. Earliest + latest `memory/YYYY-MM-DD*.md` files.
2. `MEMORY.md` for long-term milestones.
3. Session inventory + session histories (main and sub-agent sessions when accessible).
4. Optional command logs (for example `~/.openclaw/logs/commands.log`) when present.
5. Current conversation/tool logs.

## Collection Workflow

1. Identify a provisional birth marker from earliest evidence.
2. Enumerate all visible sessions.
3. Collect session history with pagination/chunking up to API limits.
4. Detect and record truncation signals (`truncated`, `droppedMessages`, content truncation, hard API limits).
5. Normalize entries into: `timestamp`, `action`, `result`, `source`, `evidence_level`.
6. De-duplicate overlaps across memory/session/log sources.
7. Sort strictly oldest → newest.
8. Tag each row:
   - **Confirmed**: direct evidence in source.
   - **Inferred**: indirect reconstruction from partial evidence.
9. Compute coverage and confidence.

## Full vs Partial Decision

Set one status only:

- **FULL**: no known missing windows in accessible retention scope, and no unresolved truncation affecting material periods.
- **PARTIAL**: any known inaccessible scope, missing windows, capped/truncated history, disabled logging, or uncertain period.

If status is PARTIAL, explicitly state what could not be seen.

## Output Format (Required)

- **History status:** FULL | PARTIAL
- **Birth marker:** first known timestamp + source
- **Coverage window:** `coverage_start` → `coverage_end`
- **Sources checked:** each source + status (`ok`, `missing`, `inaccessible`, `truncated`)
- **Missing ranges:** explicit date/session windows not fully visible
- **Chronological history (oldest → newest):**
  - `YYYY-MM-DD HH:MM (TZ) — [Confirmed|Inferred] Action — Result`
  - Include source pointer when possible (file path + line, session key, or log reference)
- **Gaps / limits:** concise list of what prevented full reconstruction
- **Confidence:** high / medium / low + one-line reason

## Fail-Safe Wording

When full lifetime cannot be guaranteed, use wording equivalent to:

> “I can provide a **PARTIAL** lifetime history based on visible records. Some periods are not accessible in this runtime.”

Never state or imply “complete from birth to now” in PARTIAL mode.

## Quality Rules

- Prefer verifiable operational actions over broad narrative summaries.
- Keep chronology strict; if exact time is unknown, use best-known granularity and label uncertainty.
- If user asked for “all actions,” include both milestones and notable operational/tool actions.
- Keep the report audit-friendly: compact, timestamped, source-backed.
