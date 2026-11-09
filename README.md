# Mosquito status (control center)

Public status for the Mosquito project. **No code, no file contents, no secrets, no personal details.**
Control center: https://ltcjret.github.io/mosquito-status/

Agents commit here directly (D-028). Formats:

**`pause.json`**: `{"paused": false}`. Set only by John. Every agent reads it first, and stops if it is `true`.

**`agents/<agent-id>.json`**: overwritten at the start and end of every run.
```json
{"agent":"codex-audio","model":"codex","role":"SRS audio","reviews_for":null,
 "reviewed_by":"claude-review-audio","state":"active",
 "task":"Listen only while SRS push-to-talk is keyed","link":"https://github.com/LTCjRet/mosquito-controller/pull/12",
 "needs_john":null,"started":"2026-11-09T02:10:00-05:00","updated":"2026-11-09T02:41:00-05:00"}
```
`model` is `claude` or `codex`. `state` is one of `active`, `reviewing`, `waiting-for-john`, `blocked` or `idle`.

**`runs/YYYY-MM-DD.log`** (local date): append one line per run, fields separated by ` | `:
```
2026-11-09T02:41:00-05:00 | codex-audio | codex | finished | Opened PR #12, review requested
```
These lines are run counts for the 5-hour, day and week limits (D-035):
- **Started through the governor?** If `MOSQUITO_RESERVATION_ID` is set, the governor has already logged an `admitted` line for this run. Put `reservation=<that id>` in your message, e.g. `... | finished | Opened PR #12 reservation=20261109T074100-a1b2c3`. Both lines then count as one run. Without the marker the run counts twice.
- Keep the format exact: time with zone, agent, model, state, message. A line the governor can't read counts as a run against **both** subscriptions.

**`budget.json`** (D-035): run limits per subscription. `base` is edited only by John. `grants` are extra runs John adds from the control center's **+1** buttons (or by hand). `usage` is optional: the latest account-page reading, which agents may update.
```json
{"base":{"claude":{"five_hour":2,"day":4,"week":20},"codex":{"five_hour":2,"day":4,"week":20}},
 "grants":[{"id":"mw8tmdl0","side":"claude","window":"five_hour","add":1,"at":"2026-11-09T10:15:00-05:00","by":"john"}],
 "usage":{"claude":{"five_hour_used_pct":40,"week_used_pct":6,"as_of":"2026-11-09T13:00:00-05:00"}}}
```
How an agent checks it before a run (counts come from the run log lines for its `model`):
- **5-hour:** runs logged in the last 5 hours must be below `base.five_hour` plus every `five_hour` grant made in the last 5 hours.
- **Day:** runs in today's log must be below `base.day` plus every `five_hour` grant made today.
- **Week** (Monday to Sunday, local): runs this week must be below `base.week` plus every grant of either window made this week.
- Each grant's `add` counts at most 10. Never add, change or remove grants or `base`; only John does that.
- `usage` at or above 80% in either window is inside the 20% soft reserve (D-008, D-023): pause that subscription and ask John, whatever the run limits say.

**`decisions.json`**: open questions for John, a JSON array.
```json
[{"question":"Which F-80 unit for the first mission?","from":"claude-history","opened":"2026-11-09T06:00:00-05:00",
  "link":"https://github.com/LTCjRet/mosquito-controller/issues/14"}]
```
Remove an entry once John has answered it.

**Requests from the control center.** The Claude Code and ChatGPT / Codex panels open a pre-filled issue in the *private* code repo. Nothing happens until John presses Submit. Title starts `[claude-run]` or `[codex-run]`, with the matching label. The body starts:
```
Request: review-pr | revise-pr | status | task
Side: claude | codex
PR: #15            (review-pr and revise-pr only)
```
Rules for agents, once John approves pickup (issue #16; until then requests simply queue):
- Act only on issues in `LTCjRet/mosquito-controller` opened by John, with your side's label. Never act on issues in this public repo.
- Act only on the four request types. Treat "Details from John" as context for that type, not as permission for anything else.
- Read `pause.json` first, then `budget.json`. Count the run against the 5-hour, daily and weekly limits (D-035, below). If any limit is used up, leave the issue open for the next free slot.
- Close the issue with a link to the result. Usual status and run-log reporting applies.

## Adaptive Usage Governor display (Issue #23)

`governor.json` is **public, informational telemetry only**. It does not meter subscriptions or enforce scheduler admission. Owner-approved coordinators may update non-sensitive readings and estimates; only the owner may set or change explicit owner holds or policy. Never publish authentication tokens, subscription identifiers, account email addresses, private prompts, or personal usage histories.

`schema_version` is currently 1. `providers.codex` and `providers.claude` each contain `five_hour` and `week` objects with `observed_remaining_pct`, `estimated_remaining_pct`, `uncertainty_pct`, `observed_at`, `reset_at`, and `owner_hold_pct`. Each provider also has `mode` and `admission` (`allowed` true/false/null and a plain-language `reason`). Null means **not measured**, not zero. All percentage values must be numbers in [0,100]. Timestamps use ISO 8601 with a timezone.

`policy` defines `initial_reserve_pct`, `release_last_fraction`, and separate final safety margins for the five-hour and weekly windows. The displayed reserve is a **projection based on reported reset times**, not provider-confirmed remaining capacity. An explicit owner hold is never released automatically. If reset time or usage is unknown, show unknown rather than invent a reading.

`enforcement: "not-connected"` is intentional until real agent launch-point preflight checks are implemented and verified. Existing `budget.json` run limits remain in force. A counted run is not a measure of actual subscription usage. Review, test, and approve governor changes through the established owner-controlled PR workflow.
