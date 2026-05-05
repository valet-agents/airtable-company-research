This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## Setup

### Connectors

- **airtable-mcp**: The Airtable MCP server, OAuth-authenticated. The agent uses it to discover the Accounts table, list rows in `createdTime` order to detect new arrivals, read fields for ad-hoc Q&A, and (when explicitly asked in Slack) update record fields. Add it from the catalog at the org level so other Airtable-powered agents can share it.
- **parallel-search-mcp**: Parallel's deep-research search MCP. The agent uses it to research each newly-added account — tech stack, recent funding, key contacts — with source URLs returned alongside every result. Keyless; add it from the catalog at the org level.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts each new-account dossier to whichever channels the bot has been invited to. Slack writes use the auto-injected outbound Slack connector.
- **heartbeat** (heartbeat): Fires every 5 minutes to sweep the Accounts table for newly-added rows. Declared inline in `valet.yaml`, so it's created automatically by the dashboard setup flow.

### Secrets

This agent uses the OAuth variant of the Airtable MCP and the keyless Parallel MCP, so no API tokens are needed at the org or agent level. The Airtable OAuth grant happens in the dashboard setup flow when you connect Airtable.

### External Setup

1. After deploy, complete the Airtable OAuth grant in the dashboard. Authorize the workspace and base that contains your Accounts (or Companies) table.
2. Optional but recommended: set `ACCOUNTS_BASE_ID` and `ACCOUNTS_TABLE_ID` (or `ACCOUNTS_TABLE_NAME`) env vars on the agent so the heartbeat skips the discovery step. Without them, the agent picks the first table it finds named `Accounts`, `Companies`, `Customers`, or `Prospects`.
3. Invite the agent's Slack bot to whichever channel(s) you want new-account dossiers in. The agent posts to every channel it's a member of — invite it to one focused channel, or several. If the bot has not been invited anywhere, the dossier is sent as a DM to the workspace install user with a one-line nudge to invite it somewhere.
4. Invite the bot to any additional channels where teammates should be able to @mention it for ad-hoc company research (e.g. a sales channel for *"tell me about Vela Robotics — what's their stack?"* follow-ups).
5. The first heartbeat fire after deploy seeds the watermark to the most recent existing row — it does not dossier the backlog. Add a new row to the Accounts table to smoke-test the flow, or @mention the bot in Slack with a research question to exercise the Slack + Parallel path immediately.

## Customizing

- **Change the heartbeat cadence**: edit the `every` value on the `heartbeat` channel in `valet.yaml` (e.g. `1m` for low-volume teams that want near-real-time, `15m` for high-volume teams that want batched dossiers), then redeploy.
- **Pin the table explicitly**: set `ACCOUNTS_BASE_ID` + `ACCOUNTS_TABLE_ID` (preferred) or `ACCOUNTS_TABLE_NAME` on the agent to skip discovery and avoid ambiguity when multiple tables match.
- **Tune dossier sections**: edit the *Phase 4: Compose the dossier* template in `SOUL.md` to add or drop sections (e.g. add a `*Hiring*` block, drop `*Key contacts*` if you only care about firmographics).
- **Cap dossiers per fire**: change the `5 per fire` cap in *Phase 2* of the SOUL workflow to bound research cost on bursty days.
