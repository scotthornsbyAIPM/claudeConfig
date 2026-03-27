---
name: shared-fireflies
description: Fireflies meeting assistant. List recent meetings or search across all meetings, then view full details including summary, key topics, and action items (Scott's highlighted separately). Use when the user runs /shared-fireflies.
---

# Fireflies Meeting Assistant

Invoked via `/shared-fireflies`. Provides access to meeting transcripts via the Fireflies MCP.

## Entry Point

Present this menu when the skill starts:

```
Fireflies Meeting Assistant
───────────────────────────
1. List recent meetings (last 30 days)
2. Search meetings
```

Wait for the user to type 1 or 2.

---

## Option 1 — List Recent Meetings

Call `mcp__fireflies__fireflies_get_transcripts` with:
- `limit: 50`
- No date filters (last 30 days worth — calculate `from_date` as today minus 30 days in YYYY-MM-DD format)

Display a numbered list in this format:

```
 #  Title                            Date        Attendees
 1  Risk Mapping                     27 Mar      Scott, Rob, Matt
 2  Requirements (session 2)         25 Mar      Scott + 8 others
 3  Claude Dev Process               21 Mar      Scott, Lee, Siva...
```

Rules for attendees column:
- Extract first names only from email addresses (e.g. `scott@asset-intelligence.com` → `Scott`)
- If 3 or fewer attendees, list all first names
- If more than 3, list first 3 then `+ N others`

Prompt: `Select a meeting number (or Q to quit):`

---

## Option 2 — Search Meetings

Prompt: `Search for:`

Call `mcp__fireflies__fireflies_search_transcripts` with the user's input as the query.

Display results in the same numbered list format as Option 1.

If no results: `No meetings found for "[query]". Press Enter to go back.`

---

## Meeting Detail View

Once a meeting is selected, call `mcp__fireflies__fireflies_get_transcript_details` with the meeting ID using parameter name `transcript_id`. Fetch and store the data but **do not display the detail view automatically**.

Instead, go straight to the meeting actions menu, showing just the meeting title as context:

```
[Meeting Title]
───────────────────────────────
1. Export transcript
2. Export summary
3. Print summary
4. Ask questions about this meeting
5. View my action items
[B] Back to list   [Q] Quit
```

---

## Meeting Actions

### 1. Export Transcript

Ask: `Format? [T] Plain text  [M] Markdown`

Fetch the full transcript sentences from the already-fetched transcript detail data. Write to the current working directory (root) as:
- `[Meeting Title] - Transcript.[txt|md]` (sanitise title for filename — remove special chars)

Format the transcript with speaker names and their lines. For markdown, use `**Speaker:**` formatting.

Confirm: `Saved: [filename]`

### 2. Export Summary

Ask: `Format? [T] Plain text  [M] Markdown  [D] Word doc`

Compile the summary in the same format as the detail view (overview, key topics, all action items, keywords). Write to root as:
- `[Meeting Title] - Summary.[txt|md]`
- For Word doc: pass the markdown content through the `md-to-word` skill with brand "Superbia"

Confirm: `Saved: [filename]`

### 3. Print Summary

Display the full meeting detail inline:

```
[Title] — [Date DD Mon YYYY] ([duration] mins)
Attendees: [first names of all participants]

OVERVIEW
--------
[summary.overview]

KEY TOPICS
----------
[summary.shorthand_bullet — strip emoji, show as clean bullet points]

SCOTT'S ACTION ITEMS
--------------------
[Lines from summary.action_items assigned to "Scott Hornsby" — if none, show "None recorded"]

ALL OTHER ACTION ITEMS
----------------------
[All remaining action item lines grouped by person]

KEYWORDS
--------
[summary.keywords joined with commas]
```

### 4. Ask Questions About This Meeting

Display: `Ask a question about this meeting (or "done" to go back):`

Use the already-fetched transcript detail data to answer the question. Stay in a loop — after each answer, prompt again — until the user types `done`.

### 5. View My Action Items

Display only Scott's action items:

```
YOUR ACTION ITEMS — [Meeting Title]
------------------------------------
- [action item]
- [action item]
```

If none: `No action items recorded for you in this meeting.`

Then show: `[P] Print  [E] Export  [B] Back`

- P → display inline again
- E → ask format (txt or md), save to root as `[Meeting Title] - My Action Items.[txt|md]`
- B → return to meeting actions menu

---

After any action completes, return to the meeting actions menu (except where noted above).

- B → return to the previous list (re-use the same results, don't re-fetch)
- Q → end the skill

---

## Notes

- Scott's email is `scott@asset-intelligence.com` — use this to identify his action items
- Dates from the API are Unix timestamps in milliseconds — convert to readable format (e.g. `27 Mar 2026`)
- Duration from the API is in minutes as a float — round to nearest whole number
- Always use `mine: false` behaviour (already patched in the local MCP server) so all attended meetings are visible, not just ones Scott recorded
- The list call returns metadata only (id, title, date, duration, participants) — full summary and sentences are only fetched when a meeting is selected
- When parsing participants, deduplicate and extract first name from email (e.g. `scott@asset-intelligence.com` → `Scott`, `john.cafferty@furnleyhouse.co.uk` → `John`)
- Use `PYTHONIOENCODING=utf-8` when running Python via Bash to avoid Windows encoding errors with special characters
