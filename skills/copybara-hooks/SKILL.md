---
name: copybara-hooks
description: Set up, edit, test and troubleshoot Copybara clip hooks — commands that run automatically when a clip arrives in a stream. Use when the user wants something to happen when they post a clip ("run this script when I drop a zip in X", "why didn't my hook fire?", "add a hook that…", "set up the watch agent"), or asks about ~/.config/copybara/hooks.toml, `cb hook run/watch/queue/log`, or the background hook agent.
---

A **hook** runs a command when a new clip matches a filter. The background agent
(`copybara hook watch`) notices new clips and fires them.

Config: `~/.config/copybara/hooks.toml` (override with `COPYBARA_HOOKS_CONFIG`).
The CLI is `copybara`, aliased `cb`. Check it exists first — `cb version`; if
missing, `brew install krizpoon/tap/copybara`.

## Writing a hook

```toml
[[hooks]]
name = "design-handoff-ingest"     # required, unique; how log/queue refer to it
enabled = true                      # default true

[hooks.match]                       # all conditions must pass; omit = don't care
stream = "copybara"                 # alias, name, or UUID
types = ["file"]                    # text image file audio secret link
filenamePattern = 'handoff.*\.zip'  # regex, single-quoted (see gotchas)
textPattern = 'TODO'                # regex against the clip's text
caseInsensitive = true              # applies to both patterns

[hooks.action]
command = ["/absolute/path/script.sh", "{file}", "{filename}", "{id}"]
stdin = "none"                      # none | text | file
timeoutSeconds = 60                 # default 30
keepTempFile = false                # keep {file} after the hook returns

[activityLog]                       # optional, once per file
enabled = true
keepDays = 30                       # entries older than this are deleted
includeClipText = false             # clip text into the iCloud activity log
```

**Placeholders** in `command`: `{text}` `{file}` `{id}` `{stream}` `{kind}`
`{filename}`. The same values arrive as environment variables —
`COPYBARA_TEXT`, `COPYBARA_FILE`, `COPYBARA_CLIP_ID`, `COPYBARA_STREAM_ID`,
`COPYBARA_STREAM_NAME`, `COPYBARA_STREAM_ALIAS`, `COPYBARA_KIND`,
`COPYBARA_FORMAT`, `COPYBARA_FILENAME`, `COPYBARA_UTI`, `COPYBARA_TEXT_FILE`,
`COPYBARA_CREATED_AT`, `COPYBARA_HOOK`.

`{file}` is a temp download, **deleted as soon as the hook returns** — copy it
if the work outlives the command, or set `keepTempFile = true`.

## Setting up the agent

```bash
brew services start krizpoon/tap/copybara     # runs `copybara hook watch`
brew services restart krizpoon/tap/copybara   # after ANY config edit
```

Log: `~/Library/Logs/copybara/watch.log` (self-rotating at 1 MB, one archive).

## Verifying — always do this, don't assume

1. `cb hook run` — one pass, fires what's due, exits. Best first test.
2. Post something that matches, then `cb hook log` — shows each firing with
   exit code and duration. `cb hook log --failed` for just the failures.
3. `cb hook queue` — pending and failed work. `cb hook queue retry ID` or
   `--all` to re-run.

Exit codes: 0 ok, 1 usage, 2 not found, 3 CloudKit/network.

## Gotchas that actually bite

**Config is read once, at agent start.** Editing `hooks.toml` does nothing until
`brew services restart krizpoon/tap/copybara`. This is the most common "my hook
doesn't fire".

**A hook whose `match.stream` doesn't exist is dropped silently.** Resolve the
name first with `cb stream ls` and check it matches.

**With no enabled hooks the agent exits** ("Nothing to watch") rather than
idling — so `brew services` will show it stopped.

**Use absolute paths in `command`.** The agent runs under launchd with a minimal
PATH; a bare `node`, `python3` or `gh` fails with `env: node: No such file or
directory` and exit 127. This has happened here before. Either give the
interpreter's full path or have the script set PATH itself.

**Single-quote regexes.** The TOML reader is minimal; in `"double quotes"` a
backslash is an escape, so `'handoff.*\.zip'` is right and `"handoff.*\.zip"`
is not.

**The TOML parser is not a full TOML implementation.** It handles comments,
`[tables]`, `[[arrays of tables]]` with dotted sub-tables, and string / int /
bool / string-array values. No inline tables (`{ a = 1 }`), no multi-line
strings, no floats, no dates.

**Adding a hook does not replay old clips.** On the agent's first run every
existing clip is recorded as `baseline` and skipped by design. Test by posting a
*new* clip.

**Latency is the poll, not the push.** Silent pushes to a background agent are
throttled hard by macOS — measured at roughly 870 polls to 32 pushes over a
week. The fallback poll (60s default, `--interval` to change) is the real
budget, so a clip fires on average half an interval late. Don't promise
instant.

**Dev and Production are separate.** A Homebrew/notarized `cb` reads CloudKit
Production; a Debug build reads Development — different clips, different
activity log. Check which binary ran before concluding something is missing.

## Troubleshooting order

1. `cb hook log --failed` — did it fire and fail, or never fire?
2. `tail -50 ~/Library/Logs/copybara/watch.log` — is the agent alive and
   draining? Look for "Push watch active for N hook(s)".
3. `brew services list | grep copybara` — is it even running?
4. `cb hook run --hook NAME` — take the agent out of the picture entirely.
5. Run the command by hand with the same argv the hook uses; most failures are
   PATH or permissions in the script, not in Copybara.

## Rules

- **Never write a hook that deletes or overwrites the user's data** without
  saying plainly what it will do and getting a yes first. A hook fires
  unattended, on every matching clip.
- Show the user the TOML you are about to add and where it goes before writing
  it.
- After editing, restart the agent and verify with a real clip — an unverified
  hook is not a working hook.
