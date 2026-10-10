# CONFIG block (template)

The real values are **not** in this repo. They live at the top of the scheduled task's prompt, in this exact
format, followed by everything below `---` in `agent/sync-instructions.md`. The sync agent copies the page-facing
values into the database doc `meta/config` on every run.

```
CONFIG (household-specific values; {{NAME}} in the instructions below means the value here):
STORE_ID=<ICA store number, e.g. from https://handlaprivatkund.ica.se/stores/<STORE_ID>>
STORE_NAME=<store name as shown on ICA>
OWNER_NAME=<first name ICA greets the account holder with>
ARTIFACT_URL=<https://claude.ai/artifact/... — the page; the database lives here>
TASK_ID=<trig_... — the scheduled task's id>
DEVICE_NAME=<computer name of the PC whose Claude app holds the ICA login>
DEPLOYED_FROM=<short sha + subject of the repo commit the instructions were deployed from>
```
