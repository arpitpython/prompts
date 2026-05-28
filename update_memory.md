Update project memory based on this session's conversation. The goal: a future session should be able to resume instantly with full context — no ambiguity about what was decided, built, or what comes next.

## Steps

1. Read `MEMORY.md` in the project memory directory to get the current index.
2. Read every memory file listed there (they are small — required to avoid duplicates and stale overwrites).
3. Review the **entire conversation** from this session. Extract everything new or changed.
4. Update existing files or create new ones for completed work, decisions, and constraints (see rules below).
5. **Always overwrite `session_last.md`** with a fresh structured record of this session (format below). This is mandatory — never skip it.
6. Refresh `MEMORY.md` so every file has a current one-line entry (including `session_last.md`).

---

## Rules for updating existing memory files

- **Remove stale embedded "Next step" or "Next action" notes** from inside project bullets before writing new content. These rot and cause the wrong next step to surface in future sessions.
- **Do not embed next step inside a project bullet** — the next step lives only in `session_last.md`.
- For completed work: mark it COMPLETE with date; remove any "⬅️ NEXT" or "NOT STARTED" markers from it.
- For the sheet/module that is now the next target: add `⬅️ NEXT` marker to its bullet so it is visually unambiguous.
- Never duplicate what is already in CLAUDE.md.
- Capture feedback / constraints / architectural decisions in their own typed memory files (feedback_*.md, reference_*.md).

## What to capture

- **Completed work** — what was built or fixed, which file/module, key design decision (not code mechanics).
- **Decisions and constraints** — anything that affects future development: API quirks, business rules, external system behaviour, approved plans.
- **Blockers and open questions** — anything explicitly deferred or awaiting external input.
- **Feedback corrections** — if the user corrected an approach, save it as a feedback_*.md memory.

## What to skip

- Try/catch boilerplate and standard library usage — obvious from the code.
- Failed experiments unless the lesson matters (e.g. "never retry X because Y").
- Conversation filler and anything that won't help a future session make better decisions.

---

## session_last.md — mandatory, always overwrite

Location: same memory directory as all other memory files.
Format (use exactly these section headers):

```markdown
---
name: Last session record
description: Structured record of the most recent session — what was discussed, built, decided, and what comes next
type: project
---

## Session date
YYYY-MM-DD

## What we discussed
- Bullet list of every topic, question, or issue raised this session (not just completed work — include analysis, debugging, decisions, plan reviews)

## What was built / fixed
- Module or file name — one sentence on what it does or what was fixed
- (repeat per item; omit section if nothing was built)

## Key decisions made
- Decision — why it was made (constraint, user preference, external system behaviour)
- (repeat per decision)

## Open questions / blockers
- Item — why it is blocked or deferred
- (omit section if none)

## Next step
One specific action to take at the start of the next session. Must be specific enough to begin immediately — name the exact script, command, sheet, or file. No vague "continue" statements.
```

---

## After updating

Reply with:
- Bullet list of what changed (new / updated / removed), one line each.
- The next step, copied verbatim from `session_last.md`.
