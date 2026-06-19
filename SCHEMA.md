# StickFix `--json` output schema

Put a global flag **before** the sub-command:

```
StickFix [--json] [--quiet] [-y/--yes] <command> [args...]
```

- `--json` — print exactly **one** JSON object to **stdout**; all human/log text goes to
  **stderr**, so stdout stays a single parseable object. Errors also emit one object.
- `--quiet` / `-q` — suppress human output; rely on the exit code (errors still print to
  stderr). Ignored when `--json` is given (`--json` already keeps stdout clean).
- `--yes` / `-y` — accepted for scripting convenience; the CLI is already non-interactive,
  so it is a no-op today.
- `--json` / `--quiet` are rejected for the GUI (exit `2`).

## Exit codes

| code | meaning |
|---|---|
| `0` | success |
| `1` | failure — bad/non-R36/unparseable dtb, missing backup, failed check. In `--json` mode still emits `{"ok": false, "reason": "..."}` |
| `2` | usage error (bad arguments, or `--json`/`--quiet` with the GUI) |

Every object has a boolean **`ok`** and a **`command`** field. On any handled error the
object is `{"command": <cmd>, "ok": false, "reason": <message>}`.

## Per-command shapes

### `inspect <dtb>`
```json
{ "command": "inspect", "ok": true, "file": "H:\\...dtb", "sha": "ab12cd34ef56",
  "parsed": true, "model": "AISLPC R36TMax", "is_target": true,
  "driver": "odroidgo3-joypad", "mux_enabled": true,
  "led": true, "led_state": "on", "led_triggers": {"power":"default-on","charge":"none"},
  "leds": [{"name":"led-0","function":"charging","pin":17,"default_state":"off","trigger":"none","on":false}, ...],
  "deadzone": 64, "fuzz": 32, "flat": 32, "feel": "stock",
  "mapping": [1,0,3,2], "inverts": [] }
```
`led_state` is `"on"|"off"|null`; `feel` is a preset name or `"custom"`; `mapping`/`inverts`
are `null`/`[]` when absent. (`ok` is `false` only if the file isn't a readable dtb.)

### `explain <dtb>`
```json
{ "command": "explain", "ok": true, "file": "...",
  "findings": [ {"severity":"warn","symptom":"...","cause":"...","fix":"StickFix fix ... --in-place"} ] }
```
`severity` ∈ `critical|warn|info|ok`; `fix` is a runnable command string or `null`.

### `verify <dtb>`
```json
{ "command":"verify","ok":true,"file":"...",
  "checks":[ {"check":"FDT magic","ok":true,"detail":""}, ... ] }
```
### `verify --card ROOT`
```json
{ "command":"verify","ok":true,"card":"H:\\","checked":3,"files_ok":3,"changed":0,"missing":0,
  "manifest":"H:\\.stickfix\\backups\\20260619-1217","files":[{"name":"...dtb","status":"ok"}] }
```
(`ok` is the overall pass/fail boolean; `files_ok` is the count of matching files.)

### Write commands — `fix`, `deadzone`, `preset`, `mapping`, `swap`, `invert`, `direction`, `led`, `profile apply`
```json
{ "command":"fix","ok":true,"file":"...","in_place":true,
  "out_path":"...","backup":"....stickfix-bak","model":"AISLPC R36TMax",
  "sha_before":"...","sha_after":"...","changed":true,
  "changes":["added amux-en-gpios (mux enable - revives the sticks)","power LED -> off ..."] }
```
With `--dry-run` the object instead carries the preview plan and writes nothing:
```json
{ "command":"fix","dry_run":true,"ok":true,"file":"...","op":"fix","model":"...","is_target":true,
  "error":null,"noop":false,"changes":[...],"deltas":["Status LED: false -> true", ...],
  "sha_before":"...","sha_after":"...","size_before":67346,"size_after":67669,"size_delta":323 }
```

### `scan`
```json
{ "command":"scan","ok":true,
  "cards":[ {"root":"H:\\","level":"yes","distro":"AURKNIX","version":"20260101",
             "reason":"...","models":[...],"targets":[...],
             "dtbs":[{"path":"H:\\...dtb","name":"...dtb"}], "all_count":1 } ] }
```
`level` ∈ `yes` (repairable) / `boot` (handheld card, nothing to fix) — only usable cards listed.

### `preview [--root DIR]`
```json
{ "command":"preview","ok":true,"root":"H:\\",
  "files":[ {"name":"...dtb", "op":"fix","error":null,"noop":false,"changes":[...],"deltas":[...],
             "sha_before":"...","sha_after":"...","size_before":...,"size_after":...,"size_delta":...} ] }
```

### `backups [--root DIR]`
```json
{ "command":"backups","ok":true,"root":"H:\\",
  "backups":[ {"ts":"20260619-1217","dir":"...","created":"20260619-1217","count":1,
               "models":["AISLPC R36TMax"],"tool":"StickFix 4.0","op":"repair",
               "status":"ok","verified":true,
               "files":[{"name":"...dtb","model":"...","verify":"ok"}]} ] }
```
`status` ∈ `ok|corrupt`; per-file `verify` ∈ `ok|corrupt|missing|unverified`.

### `restore-card [--root DIR] [--snapshot TS]`
```json
{ "command":"restore-card","ok":true,"backup":"...","restored":["...dtb"],"skipped":[],"prerestore":"...-prerestore" }
```
`ok` is `false` if files were refused (e.g. checksum mismatch) and none restored.

### `restore <dtb>`  (single-file `.stickfix-bak`)
```json
{ "command":"restore","ok":true,"file":"...","backup":"....stickfix-bak","sha":"..." }
```

### `profile export <dtb>`
```json
{ "command":"profile-export","ok":true,"file":"...","out":"...stickfix-profile.json",
  "profile":{"schema":1,"tool":"StickFix 4.0","model":"...","target_sig":"1a2b3c4d",
             "mapping":[1,0,3,2],"inverts":["rx"],"deadzone":64,"led_power":true} }
```

---

*All sha fields are 12-char content fingerprints (`sha_before`/`sha_after` for writes, `sha`
for the read-only `inspect`/`restore`). Backup integrity uses a separate full sha256 stored
in each snapshot's `manifest.json`.*
