# Copybara Releases

Binaries for the Copybara CLI, and the Claude Code plugin that teaches an agent
to use it.

## CLI

```bash
brew install krizpoon/tap/copybara
```

`copybara` (aliased `cb`) posts text and files to your Copybara streams through
your own iCloud, so they appear on your other devices. `cb --help` lists the
commands.

## Claude Code plugin

Gives Claude two skills: posting and reading clips, and setting up **hooks** —
commands that run automatically when a clip arrives in a stream.

```
/plugin marketplace add krizpoon/copybara-releases
/plugin install copybara@copybara
```

It drives the `copybara` CLI, so install that first.

| skill | what it covers |
|---|---|
| `copybara` | streams, posting, fetching clips back out, the bin |
| `copybara-hooks` | `~/.config/copybara/hooks.toml`, the watch agent, testing and troubleshooting a hook |

Releases here are published by `scripts/publish-cli.sh` in the main repo. The
plugin is versioned separately from the CLI — it tracks the command surface, not
the binary.
