# shopping-list — Inköpslistan + ICA sync

A shared shopping list for the household that Claude syncs into the household's ICA online cart.
Household-specific values (store, names, links, ids) are not in this repo — see **Config** below.

## Repo layout

| Path | What | Live? |
|---|---|---|
| `agent/sync-instructions.md` | The sync agent's logic. | **Live** — every run clones `master` |
| `page/inkopslistan.html` | Page source (UI). | Source of truth; deployed by publishing to the artifact (see below) |
| `docs/data-model.md` | Database collections and fields. | Reference |
| `task/bootstrap.md` | Sync task settings + its bootstrap prompt. | Copy of what's in the task |
| `task/keepalive.md` | Hourly ICA login check-in (scheduled task "ICA login keepalive"). | **Live** — every check clones `master` |
| `config.example.md` | The private CONFIG block, with placeholder values. | Template |

## Parts and where they run

| Part | Where |
|---|---|
| Page | claude.ai artifact `{{ARTIFACT_URL}}` (needs claude.ai for `db`, `user`, `mcp` capabilities) |
| Database | Anthropic-hosted store attached to the artifact (see `docs/data-model.md`) |
| Agent runtime | Scheduled task "ICA shopping list sync" → fresh cloud Claude session per run, prompt = CONFIG block + bootstrap that clones this repo and follows `agent/sync-instructions.md` |
| Browser + ICA login | Claude desktop app's built-in browser on the household computer (`{{DEVICE_NAME}}`) |

## Flow

1. Household edits items/prefs on the page → DB.
2. **Synka nu** (owner): page writes `meta/request {mode:"sync"}` and books the task for the next whole minute
   (`update_trigger run_once_at`). Scheduler-started runs get the PC attached; direct `fire_trigger` from a browser doesn't.
3. Run: clone this repo → follow `agent/sync-instructions.md` → read DB → drive ICA in the desktop app's browser → write results + `runs/<id>` log (DevOps panel shows the commit used).
4. **Jag har beställt** → same with `mode:"ordered"`: reads the latest placed ICA order into `orders`.

## Changing things

- **Agent logic:** edit `agent/sync-instructions.md` (or `task/keepalive.md`), push to `master`. The next run uses it; DevOps shows the commit.
- **Page:** edit `page/inkopslistan.html`, push, then ask Claude to "publish the page from the repo" (republishes to the same artifact URL).
- **Task settings / bootstrap:** only via Claude in a chat linked to the household computer; then update `task/bootstrap.md`.

## Config

The repo uses placeholders like `{{STORE_ID}}`. The real values live in two private places:

| Where | Holds | Who reads it |
|---|---|---|
| CONFIG block at the top of the scheduled task's prompt | all values (see `config.example.md`) | the sync agent |
| `meta/config` in the page's database | storeId, storeName, ownerName, taskId, deviceName | the page (store name, cart link, sync button, DevOps) |

The task's CONFIG block is the master copy: every run rewrites `meta/config` if it differs. To change a value,
ask Claude (in a chat linked to the household computer) to update the CONFIG block; the next run updates the page.
The published page is byte-identical to `page/inkopslistan.html`.

## Security

The agent runs with automatic approval in a browser logged into the ICA account. **Anyone who can push to
`master` controls it.** Protect `master` (PRs only, no force-push), use 2FA, don't add collaborators with write
access unless intended, review every merged line, and never commit secrets or household values — use the CONFIG block.

## Costs

- Page ↔ DB traffic: no tokens.
- Tokens are spent only by sync runs and count against the Claude account's usage limits
  (a run fails with `USAGE_LIMIT_REACHED` when the limit is hit).
- Main drivers: price hunting (C4) and pref checks (C3); both are incremental via `priceCheckedAt` / `prefsCheckedAt`.
