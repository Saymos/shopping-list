# shopping-list — Inköpslistan + ICA sync

A shared shopping list for the household that Claude syncs into the ICA online cart at
{{STORE_NAME}} (store {{STORE_ID}}).

## Repo layout

| Path | What | Live? |
|---|---|---|
| `agent/sync-instructions.md` | The agent's logic (source of truth). | Deployed to the scheduled task on request |
| `page/inkopslistan.html` | Page source (UI). | Source of truth; deployed by publishing to the artifact (see below) |
| `docs/data-model.md` | Database collections and fields. | Reference |
| `task/bootstrap.md` | Scheduled task settings + an (unused) repo-bootstrap prompt. | Reference |

## Parts and where they run

| Part | Where |
|---|---|
| Page | claude.ai artifact `{{ARTIFACT_URL}}` (needs claude.ai for `db`, `user`, `mcp` capabilities) |
| Database | Anthropic-hosted store attached to the artifact (see `docs/data-model.md`) |
| Agent runtime | Scheduled task "ICA shopping list sync" → fresh cloud Claude session per run, prompt = deployed copy of `agent/sync-instructions.md` |
| Browser + ICA login | Claude desktop app's built-in browser on {{DEVICE_NAME}} |

## Flow

1. Household edits items/prefs on the page → DB.
2. **Synka nu** (owner): page writes `meta/request {mode:"sync"}` and books the task for the next whole minute
   (`update_trigger run_once_at`). Scheduler-started runs get the PC attached; direct `fire_trigger` from a browser doesn't.
3. Run: follow the deployed instructions → read DB → drive ICA in the desktop app's browser → write results + `runs/<id>` log (DevOps panel shows the commit used).
4. **Jag har beställt** → same with `mode:"ordered"`: reads the latest placed ICA order into `orders`.

## Changing things

- **Agent logic:** edit `agent/sync-instructions.md`, push, then ask Claude (in a chat linked to {{DEVICE_NAME}}) to deploy it to the task.
- **Page:** edit `page/inkopslistan.html`, push, then ask Claude to "publish the page from the repo" (republishes to the same artifact URL).
- **Task settings / bootstrap:** only via Claude in a chat linked to {{DEVICE_NAME}}; then update `task/bootstrap.md`.

## Security

The agent runs with automatic approval in a browser logged into the ICA account. **Anyone with push access
here controls it.** Keep the repo private; don't add collaborators with write access unless intended.

## Costs

- Page ↔ DB traffic: no tokens.
- Tokens are spent only by sync runs and count against the Claude account's usage limits
  (a run fails with `USAGE_LIMIT_REACHED` when the limit is hit).
- Main drivers: price hunting (C4) and pref checks (C3); both are incremental via `priceCheckedAt` / `prefsCheckedAt`.
