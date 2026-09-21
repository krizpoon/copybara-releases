---
name: copybara
description: Read and post clips with the Copybara CLI — the user's cross-device clipboard, synced through their own iCloud. Use when they want to send text or a file to another device, look at what is in a stream, fetch a clip back out, or ask "what's in my clipboard/streams", "post this to X", "save that file from Y".
---

`copybara` (aliased `cb`) reads and writes the user's Copybara streams directly
in CloudKit, so anything posted appears on their other devices. Install with
`brew install krizpoon/tap/copybara` if it is missing.

The grammar is `<noun> <verb>`. Every command takes `--json`, `--quiet` and
`--help`.

## Streams

| | |
|---|---|
| `cb stream ls` | every stream, with alias and UUID |
| `cb stream alias STREAM ALIAS` | give a stream a short name |
| `cb stream rm STREAM [--keep-clips]` | delete it; clips go to the bin, or to Main with the flag |

`--stream` anywhere accepts an **alias, a name (case-insensitive), or a UUID** —
resolve it yourself only if you need the UUID for something else.

## Clips

| | |
|---|---|
| `cb clip ls [--stream S] [--type KINDS] [--limit N]` | list clips |
| `cb clip add TEXT [--stream S]` | post text |
| `cb clip add --file PATH [--name NAME] [--stream S]` | post a file |
| `cb clip get [ID] [--out PATH] [--stream S]` | text to stdout, files to disk; default is the latest clip |
| `cb clip rm ID [--stream S]` | move a clip to the bin |
| `cb clip tail [--stream S] [--type KINDS]` | print new clips as they arrive |

`--type` takes a comma-separated list: `text image file audio secret link`.

`post` and `save` are shorthand for `clip add` and `clip get`.

`clip get ID --out DIR/` writes the attachment under its own filename; with a
path ending in a filename it uses that name instead.

## Bin and hooks

`cb bin ls`, `cb bin restore ID`, `cb bin empty` (permanent).

For hooks — running a command when a clip arrives — use the **copybara-hooks**
skill instead; it covers the config format and the background agent.

## Exit codes

`0` ok · `1` bad usage · `2` no such thing · `3` CloudKit, network or account.
Branch on these rather than on message text.

## Rules

- **Secrets stay secret.** A Secret clip's value is end-to-end encrypted;
  `cb clip get` will decrypt it. Never print one into a transcript, a log, or a
  file the user did not ask for.
- **`bin empty` is permanent** and `stream rm` moves real clips. Say what will
  be deleted and get a yes first.
- Posting is visible on every device the user owns, immediately. Treat it as
  publishing, not as a scratch file.
- Report the stream by the name the user used, not the UUID.
