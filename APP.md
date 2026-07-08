# Morning Digest — APP.md

> Read this first before working on this app. It's the single source of truth for what the app is, how it works, and where things live.

## Concept
A personal PWA for Nick with two tabs (switcher in the header):
- **Digest** — a daily "morning action digest" of actionable to-dos pulled from email, calendar, Teams, and meeting notes that are due within 48h or overdue. Nick checks items off as he completes them.
- **Recaps** — end-of-day client meeting recaps built from tl;dv transcripts. Nick reviews/edits each recap and taps "Create Outlook Draft" to push a real Outlook draft (pre-addressed to attendees) for review and send.

Installed to the iPhone home screen, opened morning and end-of-day.

## How it works (data flow)
### Digest tab
1. A scheduled skill (`morning-action-digest`) runs each morning at 8 AM and writes the day's digest as `digest-YYYY-MM-DD.json` into the OneDrive folder.
2. The PWA (`index.html`) signs in to Microsoft via MSAL, finds the most recent `digest-*.json` in that folder via Microsoft Graph, and renders it.
3. Checking items off saves state back to OneDrive:
   - `digest-state.json` — current check state.
   - `Done/digest-YYYY-MM-DD-done.json` — completed items, read by the skill the next day so done items don't reappear.
4. Unchecked items from yesterday show a "Yesterday" badge.

### Recaps tab
1. A scheduled task (`eod-meeting-recaps`, weekdays ~4 PM ET, cron `0 16 * * 1-5`) pulls today's EXTERNAL/client meetings from tl;dv (MCP connector, read-only), writes a recap per meeting, and saves `recaps-YYYY-MM-DD.json` into `Daily json files/Meeting Recap emails/` (the `RECAPS_FOLDER`). The task writes the file to the **local OneDrive-synced path** (the Microsoft MCP connector is read-only and cannot write files or create drafts — that's why the PWA does the drafting).
2. The Recaps tab finds the most recent `recaps-*.json` via Graph and renders one card per meeting (editable subject + body, attendee list).
3. "Open in Outlook" (`openInOutlook()`) opens an Outlook-on-the-web compose deeplink (`https://outlook.office.com/mail/deeplink/compose?to=…&subject=…&body=…`) in a new tab, pre-addressed with the edited subject/body. A "Copy" button is the fallback. Nothing sends automatically.
   - **Why a deeplink, not a Graph draft:** the Mely.ai tenant requires admin consent for Graph mail scopes (`Mail.ReadWrite`/`Mail.Send`) and Nick can't get admin access (confirmed 2026-06-25). The deeplink is just a URL, so it needs no mail permission. App `SCOPES` stays `User.Read` + `Files.ReadWrite`.

**Recaps JSON schema** (the task must emit exactly this):
```
{
  "date": "Thursday, June 25, 2026",
  "meetings": [
    { "id": "<tldv id>", "name": "<meeting>", "time": "2:00–2:30 PM EDT",
      "attendees": ["client@x.com"], "subject": "Recap: <meeting> — June 25, 2026",
      "body": "Hi all,\n\n...\n\nBest,\nNick" }
  ]
}
```

## Sections in a digest
`overdue`, `critical`, `today`, `tomorrow`, plus email / calendar / client / Teams groupings (see CSS classes `sh-ovr`, `sh-crit`, `sh-tod`, `sh-tmrw`, `sh-eml`, `sh-cal`, `sh-cli`, `sh-teams`).

## File map
- `index.html` — the entire app (HTML + CSS + JS in one file). Single-file PWA.
- `manifest.json` — PWA manifest (name "Morning Digest", standalone, sun icon).
- `SETUP.md` — one-time deploy + Azure setup instructions.
- `Daily json files/` — where the digest JSONs live (this is the Graph `FOLDER` target).
  - `digest-YYYY-MM-DD.json` — daily digest the app reads.
  - `digest-state.json` — saved check state.
  - `Done/` — completed-item snapshots read by the skill next day.
  - `HTML Version/` — standalone HTML digest written by the skill each day (not read by the PWA; for quick viewing). Added 2026-06-25.
  - `Meeting Recap emails/` — subfolder holding `recaps-YYYY-MM-DD.json`, the daily client-meeting recaps the Recaps tab reads. Written by the `eod-meeting-recaps` task. This is the `RECAPS_FOLDER` Graph path. Added 2026-06-25.

## Config (in index.html, ~line 232)
- `CLIENT_ID` = `6d312e2b-8697-4162-92c3-fb76ad4782da` (Azure app registration)
- `TENANT_ID` = `add1a0bc-b68d-4df0-aefd-d9999f26df0c` (Mely.ai)
- `FOLDER` = `Claude Main Folder/Apps/morning-digest-app/Daily json files` (Graph path; root folder renamed from `Claude Dashboard` → `Claude Main Folder` on 2026-06-25. The scheduled `morning-action-digest` skill writes its JSON here too — keep both in sync if the folder ever moves.)
- `RECAPS_FOLDER` = `FOLDER + '/Meeting Recap emails'` — where the Recaps tab reads `recaps-*.json` and where the `eod-meeting-recaps` task writes them. Keep the task's write path and this const in sync.
- Graph permission needed (delegated): `Files.ReadWrite` only. The Recaps tab opens emails via an Outlook web compose deeplink (a URL), so no mail permission is required. (Graph `Mail.ReadWrite` was tried first but the tenant requires admin consent Nick can't get — see Recaps data flow above.)

## Stack
Single-file vanilla HTML/CSS/JS. MSAL.js for Microsoft auth, Microsoft Graph for OneDrive file read/write. No build step, no framework. Deployed by drag-and-drop to Netlify. Dark mode via `prefers-color-scheme`.

## Decisions & conventions
- Keep it single-file — no build tooling.
- All data lives in OneDrive, not in browser storage.
- The app is read-mostly; the skill produces the data.

## Open items / ideas
- (none recorded yet — add here as they come up)

## Working agreement
Per project rules: **always ask before modifying anything.**
