---
name: ontask
description: Add tasks, reminders, homework, school items, and events to younes's OnTask list at wxundup.github.io/younesjabara-site/. Use when the user says anything like "remind me to...", "add this to my list", "put this on my calendar", "ontask", or mentions homework, a deadline, an exam, a wedding, or anything they need to get done.
---

# OnTask — younes's task list

Website: `https://wxundup.github.io/younesjabara-site/#ontask`

Agent instructions: `https://wxundup.github.io/younesjabara-site/llms.txt`

API base URL: `https://younes-ontask.woundedmc.workers.dev/ontask`. GitHub Pages serves the website only; send API requests to this separate backend.

Everything lives in ONE POST. Fill in what you know, drop what you don't.

## How to add a task

Send a POST with JSON to `https://younes-ontask.woundedmc.workers.dev/ontask/endpointforagent`:

```json
{
  "title": "text sarah about the wedding photos",
  "date": "2026-10-15",
  "time": "17:00",
  "type": "task",
  "notes": "she asked for the ones from the ceremony"
}
```

Only `title` is required. Send the POST with curl or fetch, for example:

```bash
curl -X POST https://younes-ontask.woundedmc.workers.dev/ontask/endpointforagent \
  -H "Content-Type: application/json" \
  -d '{"title": "text sarah about the wedding photos", "date": "2026-10-15", "time": "17:00", "type": "task"}'
```

## Turning what the human said into fields

- **title** — the action, short and imperative. "text sarah about the wedding photos", not "younes said I should text sarah".
- **date** — YYYY-MM-DD. If they said "tomorrow", "friday", "next friday", "today", "tonight", "oct 15", or "in 3 days", you can send the exact date or the phrase itself — the server understands those words. No date mentioned? Omit it.
- **time** — "17:00" or "5pm" or "5:30pm". "tonight" with no time is fine.
- **type** — one of:
  - `homework` — assignments, problem sets, essays, studying
  - `school` — classes, exams, quizzes
  - `event` — weddings, parties, appointments, flights, dinners
  - `task` — everything else (texting someone, pushing an update, errands). Default.
- **notes** — anything extra worth remembering.

## Simpler fallbacks (also fine)

- No-JSON raw text: `curl -X POST https://younes-ontask.woundedmc.workers.dev/ontask/endpointforagent -d "tomorrow 5pm text sarah about the wedding photos"`
- One GET: `https://younes-ontask.woundedmc.workers.dev/ontask/api/add?title=text%20sarah&date=tomorrow&time=5pm`
- If unsure how the API works, GET `https://younes-ontask.woundedmc.workers.dev/ontask/endpointforagent` — it returns plain-text instructions.

## Before adding: check for duplicates

Read `GET https://younes-ontask.woundedmc.workers.dev/ontask/api/tasks` without filters first, including completed tasks. Match titles, dates, notes, and source links. Do not recreate a completed assignment or overwrite a confirmed date. Use the connected OnTask MCP tools when available; `add_task` takes a stable source-derived ID and preserves existing completion and edits.

## Reading the list

`GET https://younes-ontask.woundedmc.workers.dev/ontask/api/tasks` → `{"ok": true, "tasks": [...]}`. Filters combine freely on one request:

- `?from=2026-10-22&to=2026-10-28` — date range. `from`/`to` also accept the words: `today`, `tomorrow`, `friday`, `oct 25`, `in 3 days`.
- `?type=homework|school|event|task`
- `?status=open|done` — `open` means not done yet.
- `?q=whitman` — semantic search over titles, notes, and dates, fused with keyword matches, best first. Combine it with the filters: `?status=open&q=quiz`.

Example — everything open in the next week: `GET https://younes-ontask.woundedmc.workers.dev/ontask/api/tasks?status=open&from=today&to=in 7 days` (URL-encode the spaces).

## Managing tasks: edit and delete

Every task in the list has an `id`. You can edit and remove, not just add:

```bash
# mark done
curl -X PATCH https://younes-ontask.woundedmc.workers.dev/ontask/api/tasks/<id> \
  -H "Content-Type: application/json" -d '{"done": true}'

# delete
curl -X DELETE https://younes-ontask.woundedmc.workers.dev/ontask/api/tasks/<id>
```

PATCH takes any task fields (`title`, `date`, `time`, `type`, `notes`, `done`). DELETE is idempotent: deleting twice is fine.

## After adding

A 200 response with `"ok": true` means the task is on the list. Tell the human what you added, including the date. If it fails, retry once, then say so.
