# New-Account Sweep (Heartbeat)

The heartbeat channel fires every 5 minutes. There is no payload
to parse — your job is to find rows added to the Airtable
Accounts table since the last sweep, research each one, and post
a dossier to Slack.

## What it does

1. Resolve the Accounts table per the SOUL **Phase 1** rules:
   prefer `AIRTABLE_BASE_ID` + `AIRTABLE_TABLE_NAME` env vars;
   otherwise auto-discover the first table named
   `Accounts`/`Companies`/`Customers`/`Prospects`. Cache the
   resolved IDs into MEMORY.md.
2. Read MEMORY.md for `last_seen_record_id` and
   `last_seen_created_time`. List rows in the table sorted by
   `createdTime` descending and take any with `createdTime` newer
   than the watermark, capped at 5 per fire.
3. For each new row (oldest first), run a Parallel deep-research
   pass per SOUL **Phase 3** — tech stack, recent funding, key
   contacts, one-line elevator pitch.
4. Compose a dossier per SOUL **Phase 4** — Slack `mrkdwn`,
   under 2,000 chars, three lines per section, every claim
   sourced.
5. Resolve target channels per the SOUL **Where to post** rules
   and post one message per channel per company. If a post fails
   for a particular channel, log and continue with the others —
   do not retry.
6. Update MEMORY.md to the newest record processed (record id +
   `createdTime`). The next fire reads from there.

## MEMORY.md state shape

The agent persists a small block in MEMORY.md to track what's
been processed. Shape:

```
## airtable-company-research

accounts_base_id: appXXXXXXXXXXXXXX
accounts_table_id: tblXXXXXXXXXXXXXX
accounts_table_name: Accounts
last_seen_record_id: recXXXXXXXXXXXXXX
last_seen_created_time: 2026-05-05T17:42:00.000Z
```

Update this block in place each fire. Do not append a new block
per fire.

## Where to post

Per SOUL: every Slack channel the bot is a member of, one
message per company per channel. If the bot is in zero channels,
DM the workspace install user with the dossier and a one-line
invite hint.

## Skip conditions

Skip posting (and stop silently) if any of these are true:

- The Accounts table can't be resolved (no env vars set and no
  table named `Accounts`/`Companies`/`Customers`/`Prospects`
  found). Write `accounts_table: not_found` to MEMORY.md and
  stop. Do not post.
- This is the first run after deploy. Seed the watermark to the
  most-recent existing row and stop — the backlog is not
  dossiered.
- Zero rows have `createdTime` newer than the watermark. Stay
  silent — no "no new accounts" post.
- A row's company name field is empty. Skip that row, log it to
  MEMORY.md, and advance the watermark past it so it isn't
  retried forever.
