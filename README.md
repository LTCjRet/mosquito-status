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

**`decisions.json`**: open questions for John, a JSON array.
```json
[{"question":"Which F-80 unit for the first mission?","from":"claude-history","opened":"2026-11-09T06:00:00-05:00",
  "link":"https://github.com/LTCjRet/mosquito-controller/issues/14"}]
```
Remove an entry once John has answered it.
