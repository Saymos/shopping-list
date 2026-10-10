# Data model (artifact DB)

Document store attached to the artifact. Collections are flat; ids are client-generated unless noted.
Limits: 25,000 docs per artifact, 256 KiB per doc. Default access: anyone with Contributor access reads/writes.

## items/{id}
List entries. Written by the page (household) and the agent.

| Field | Type | Who | Meaning |
|---|---|---|---|
| name | string | page | Text as typed (any language). Never rewritten. |
| qty | number | page | Parsed leading quantity. |
| status | `new` \| `in_cart` \| `not_found` \| `ordered` | both | Lifecycle. `ordered` items are hidden on the page. |
| lines | [{product, qty, price}] | agent | Cart products for this item; price = line total (kr). |
| product, price | string, number | agent | Legacy/compat: joined product names, sum of line prices. |
| reason | string (sv) | agent | Why this product was chosen. |
| alternatives | [{product, price, note}] | agent | Up to 3 suggestions. |
| swaps | [{from, to\|null, qty}] | page | Pending household swaps; `to:null` = remove that many of `from`. |
| adds | [{product, qty, price}] | page | Pending "buy in addition" picks from alternatives. |
| qtyChanges | [{product, from, qty}] | page | Pending quantity changes (0 = remove). |
| removeRequested | bool | page | Remove whole item from cart at next sync. |
| replacedBy / replacesId | id | page | Edit-from-cart pairing (old ↔ new item). |
| editSaved | bool | page | Replacement copy was saved (vs. abandoned). |
| sortAt | ISO string | page | Ordering key; copies get `<original>~` to sit right after. |
| syncNote | string (sv) | agent | Shown when status is `not_found` or a pref can't be met. |
| prefsCheckedAt, priceCheckedAt | ISO | agent | Incremental-work timestamps. |
| addedBy, addedAt, orderedAt | user id, ISO, ISO | page/agent | |

## prefs/{id}
`{text, strict, fromLearning, addedBy, addedAt, updatedAt}` — `strict:true` = "Alltid", otherwise "Helst".

## learnings/{id}
`{text, evidence, suggestedPref, createdAt, updatedAt}` — agent-written; page can promote to a pref or delete.

## orders/{id}
`{orderedAt, orderNumber, delivery, total, source:"ica"|"list", items:[{name, qty, product, price}], diffs:[string]}` — created only by the agent in `ordered` mode. Never modified.

## runs/{YYYYMMDD-HHMMSS}
`{startedAt, finishedAt, status:"running"|"done"|"failed", mode, trigger, sessionId, device, cartTotal, summary, error, steps:[{t, step, ok, detail}], changes:[...]}` — max 20 kept.

## meta/*
- `meta/request` `{mode, requestedAt, runAt, status, requestedBy, handledAt, runId}` — what the next run should do.
- `meta/sync` `{at, summary, runId, changes}` — last run summary shown on the page.
- `meta/prefs` `{changedAt, changedBy}` — bumped on any pref change; drives C3 skipping.
- `meta/session` `{checkedAt, loggedIn, restored, detail, lastLoggedInAt, loggedOutSince, history:[{t, loggedIn, restored}] (max 48)}` — ICA login state, written hourly by the keepalive task (`task/keepalive.md`). The page shows a banner while `loggedIn` is false.
- `meta/config` `{storeId, storeName, ownerName, taskId, deviceName, updatedAt}` — household-specific values the page needs. Written only by the sync agent from the task's CONFIG block (master copy).

## change entry
`{type:"added"|"removed"|"swapped"|"replaced"|"not_found"|"updated", item, product?, from?, to?, qty?, price?, note?}`
