---
name: fork-pane
description: Fork the current Claude session and open the fork in a new tmux side pane — one command, no extra steps
allowed-tools:
  - Bash
---

Fork this Claude session and open the fork in a new tmux side pane.

**Step 1: Parse arguments**

`$ARGUMENTS` has the form: `[<session-id>] [<name>]`

- The first token, if it looks like a UUID, is the **session ID**
- The remaining tokens (or all of `$ARGUMENTS` if no UUID was found) are the **window name**
- Both are optional

If no session ID is found in `$ARGUMENTS`, find it via:

```bash
ls -t ~/.claude/session-env/ | head -1
```

If no window name is found in `$ARGUMENTS`, generate a random human-readable name in the style of `<adjective>-<noun>` (e.g. `silent-river`, `bold-fox`, `calm-spark`). Pick words at random — do not use a fixed list.

**Step 2: Open a forked session in a new tmux pane**

Use the Bash tool to run:

```bash
tmux split-window -d -h -t "$TMUX_PANE" "claude -r <session-id> --fork-session -n '<name>'"
```

The `--fork-session` flag resumes the current session AND creates a new independent branch of it. This means both panes start from the same conversation state and can diverge independently.

**Step 3: Name the tmux window (if a name was provided)**

If a window name was parsed from `$ARGUMENTS`, rename the current tmux window to that name:

```bash
tmux rename-window "<name>"
```

This lets you navigate back to the fork later with `tmux select-window -t <name>` or `Ctrl+b '`.

**Step 4: Confirm**

Tell the user:
- The fork is now open in the right pane (new independent branch)
- The current pane is the original, unchanged
- Both start from the same conversation state
- How to switch: `Ctrl+b →` / `Ctrl+b ←`
- If a name was given: the window is named `<name>` — switch to it anytime with `Ctrl+b '` and type the name, or `tmux select-window -t <name>`
