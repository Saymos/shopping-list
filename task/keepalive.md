# Scheduled task: "ICA login keepalive"

Hourly check-in that keeps the ICA web login alive and measures how long it lasts. It never shops.

- Schedule: every 30 min via two identical tasks ("ICA login keepalive" at :51, "ICA login keepalive (:21)" at :21) — the scheduler allows at most one run per hour per task
- Requires computer: `{{DEVICE_NAME}}` (built-in browser holds the ICA login)
- Notifications: none — the result is shown on the page (banner when logged out) and in DevOps
- Writes only `meta/session` (see `docs/data-model.md`)
- Task prompt = CONFIG block (STORE_ID, OWNER_NAME, ARTIFACT_URL) + the text below `---`, deployed on request
  (loading it from GitHub at run time gets blocked by the automatic-run safety check — see `task/bootstrap.md`).

---

ICA LOGIN KEEPALIVE — a tiny check. Finish fast with as few tool calls and tokens as possible; no narration. {{NAME}} means the CONFIG value above.

1. Load the browser tools: ToolSearch query "mcp__remote-devices__Claude_Browser__" (max_results 64). If none appear (and there is an enable__mcp__remote-devices__Claude_Browser tool, call it first), wait 15 s (bash `sleep 15`) and retry, up to 3 attempts. Also load ArtifactData if deferred. If the browser tools never appear, write meta/session (step 5) with loggedIn null and detail "browser unreachable: <exact error>", then stop.
2. Don't collide with a sync: read ArtifactData url {{ARTIFACT_URL}} collection "runs" (query order_by startedAt desc, limit 1). If that run has status "running" and started less than 20 minutes ago, end with "skipped: sync running" without writing anything.
3. preview_start https://handlaprivatkund.ica.se/stores/{{STORE_ID}}/basket and get_page_text on that tab. Logged in = the page greets "Välkommen {{OWNER_NAME}}" (ICA shows the name in capitals) and has no "Logga in" header button. Logged out = "Välkommen till ICAs e-handel" or a "Logga in" header button.
4. If logged out: click "Logga in" once (find "Logga in"), wait up to 10 s, read the page. If it returns to the store logged in → loggedIn true, restored true. If it shows the ims.icagruppen.se login page ("Logga in" / "Fler inloggningssätt") → loggedIn false; leave that tab on the ICA login page and bring it to the front (tabs_select) so the person can log in right away when they open the browser. NEVER start BankID, never pick a login method, never type anything, never touch the cart. If you were logged in, close the tab you opened (tabs_close); if logged out, leave it open on the login page.
5. Write meta/session (same artifact): read it first (pin if_version; create if missing). Fields: checkedAt (now ISO, from `date -u`), loggedIn (true/false/null), restored (true only if step 4 restored it), detail (short English: what you saw), lastLoggedInAt (= checkedAt when loggedIn true, else keep), loggedOutSince (= checkedAt when loggedIn false and it wasn't already false, keep it while still false, remove it with {"__delete__": true} when loggedIn true), history: [{t, loggedIn, restored}] newest first, keep at most 48 entries.

End with one line: "ICA: logged in" / "ICA: restored" / "ICA: LOGGED OUT since <time>" / "ICA: browser unreachable".
