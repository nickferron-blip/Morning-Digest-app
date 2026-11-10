# My Control Tower — handoff

**Artifact:** https://claude.ai/artifact/CSBKXrcsBKr3douM2JFNFk
Read and write its database with the `ArtifactData` tool, passing that `url` on every call.

Built 2026-11-10 as a replacement candidate for the Morning Digest PWA. **Both run in parallel** —
nothing about the PWA, its GitHub Pages deploy, or the `morning-digest-with-client-state` cron was
changed. Nick is assessing which one wins.

---

## Why it exists

The PWA regenerated `digest-YYYY-MM-DD.json` every morning, so tasks were re-emitted daily rather
than persisting. Five of its seven OneDrive files existed only to paper over that: per-date
overrides, a completed-log matched by title (ids drifted), separate subtask state, and an edits
overlay. The skill carried ~330 lines of carry-forward rules, ID-stability audits and dedupe passes
for the same reason.

**Here a task is a durable row.** Created once, edited in place, gone when deleted or completed.
Carry-forward isn't a rule, it's the default. All of that machinery is unnecessary.

## Data model (`db` capability, declared with `user`)

| Collection | Owner | Notes |
|---|---|---|
| `tasks` | **Nick edits by hand — never overwrite** | id, title, body, source (`recap`/`flag`/`manual`), ref{subject,on,from,to,messageId,webLink}, client, clientName, status (`""`/`doing`/`waiting`/`blocked`), quadrant (`deadline`/`todo`/`delegate`/`eliminate`/`quick`), mit, durationMin, dueDate, subtasks[{id,text,evidence,done}], done, createdAt, completedAt |
| `clients` | sync job, safe to overwrite | id (`cli-<slug>`), name, owner, domain, industry, location, volume, health (`ok`/`watch`/`risk`), currentStatus, nextStep, lastActivity |
| `meetings` | sync job | id, title, startsAt, time, detail, clientName |
| `recaps` | sync job | id, subject, sentOn, clientName, actionCount |

Seeded 2026-11-10 from `digest-2026-11-10.json`, `clients.json`, `client-state/cli-*.json` and
`Meeting Recap emails/recaps-*.json`: 7 tasks, 31 clients, 5 meetings, 22 recaps.

## Views

- **Now** — Power Hour band, MIT, Quick wins, 4-column Eisenhower board, collapsed **Done** section.
- **Exceptions** — only accounts that are at risk, have an overdue task, or have gone quiet 14+ days.
  All 31 are searchable below. This is the main departure from the PWA, which showed every card daily.
- **Flow** — meetings ahead and recaps behind on one timeline.

## Interaction rules (learned the hard way)

- **Only the checkbox completes a task.** Everything else on a card opens the detail panel.
  An earlier build put `data-done` on the full-width title and silently completed 5 of 7 tasks in
  ten seconds. They were restored from `completedAt` timestamps + unchecked subtasks.
- Completion must stay reversible — hence the Done section.
- Drag the `⠿` handle to re-file; drop zones are the 4 quadrants, Quick wins, and the MIT slot.
- Escape inside an inline edit must `stopPropagation()`, or the document handler closes the panel.
- Calendar: 📅 on deadline cards and a Schedule row in the panel, both deep-linking to Outlook
  compose at the next Power Hour block (weekends skipped).

## Task sources

1. **Recap emails** Nick sends (`Recap:` in Sent Items) — one email = one task, his action-item
   bullets = subtasks. Drop bullets prefixed with a client name, and drop `Mely: <colleague> to…`
   (Edward/Hoang/Team) — those are Mely's work, not Nick's. Keep `Mely: Nick to…` and unprefixed.
2. **Flagged Outlook mail** — the real red flag. `flag.flagStatus === "flagged"`.
   **It is NOT in `outlook_email_search` results — only in `read_resource`.** So: list the inbox,
   then read each message and filter. `"complete"` (flag ticked off) is a skip, not a completion.
3. **Manual** — typed in the app, or written here via `ArtifactData`.

Nothing else creates a task. Not Teams, not tl;dv, not calendar, not a client `nextStep`.

## Scheduled job

`control-tower-sync`, weekdays 7:32 AM, `/Users/nicolasferron/.claude/scheduled-tasks/control-tower-sync/SKILL.md`
(procedure mirrored at `Claude Main Folder/Scheduled/control-tower-sync/SKILL.md`).
Purely additive: adds what isn't there, never touches an existing task. Writes are version-pinned —
on conflict it re-reads and skips, so Nick's edit always wins.

**Nick should hit "Run now" once** on the task page to pre-approve the Outlook connector.

## Open items

- Bulk task creation from a pasted list — works today by asking Claude with the URL above.
  An in-app paste box using the `sample` capability was offered and not yet built.
- "Schedule no later than 2 business days before the due date" from the PWA isn't ported; every
  current task is already past due, so it had nothing to act on.
- Client onboarding plan and meeting talking points are not carried over from the PWA.
- Sharing with Edward Ko is a maybe-later, explicitly deferred. If it happens, note that the Kruse
  model (one MIT, one Power Hour) is single-player — separate boards with mutual visibility, not a
  merged list.
