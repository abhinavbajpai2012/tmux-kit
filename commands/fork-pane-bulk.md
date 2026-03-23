---
name: fork-pane-bulk
description: Fork the current Claude session N times, opening each fork in its own tmux window with a randomly generated name
allowed-tools:
  - Bash
---

Fork this Claude session N times, each in a new tmux window with a unique auto-generated name.

**Step 1: Parse arguments**

`$ARGUMENTS` has the form: `<count> [<session-id>]`

- The first token is the **count** (required) — how many forks to create
- The second token, if it looks like a UUID, is the **session ID** to fork from

If no session ID is found in `$ARGUMENTS`, find it via:

```bash
ls -t ~/.claude/session-env/ | head -1
```

**Step 2: Generate names**

Generate `<count>` unique random human-readable names, each in the style of `<adjective>-<noun>` (e.g. `swift-canyon`, `amber-tide`, `hollow-pine`). All names must be distinct. Pick words at random — do not use a fixed list.

**Step 3: Create each fork**

For each fork (repeat `<count>` times), use the Bash tool to:

1. Create a new tmux window named after the generated name:

```bash
tmux new-window -n "<name>"
```

2. Split that window horizontally and launch the forked Claude session in the right pane:

```bash
tmux split-window -h -t "<name>" "claude -r <session-id> --fork-session -n '<name>'"
```

The `--fork-session` flag resumes the session AND creates a new independent branch of it. Every fork starts from the same conversation state and diverges independently.

Run each fork sequentially (one Bash call per fork) so tmux window creation is reliable.

**Step 4: Return focus to the original window**

After all forks are created, switch back to the window where the command was invoked:

```bash
tmux select-window -t !
```

**Step 5: Confirm**

Tell the user:
- How many forks were created
- The list of generated window names (one per line)
- Each fork is an independent branch starting from the same conversation state
- How to navigate: `Ctrl+b w` to see all windows, `Ctrl+b '` to jump by name, or `tmux select-window -t <name>`
