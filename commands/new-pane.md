---
name: new-pane
description: Open a fresh Claude session in a new tmux side pane with an auto-generated name
allowed-tools:
  - Bash
---

Open a brand-new Claude session in a new tmux side pane.

**Step 1: Parse arguments**

`$ARGUMENTS` has the form: `[<name>]`

- If provided, the token is the **window name** for the new pane
- It is optional

If no window name is found in `$ARGUMENTS`, generate a random human-readable name in the style of `<adjective>-<noun>` (e.g. `quiet-ember`, `swift-brook`, `iron-mist`). Pick words at random — do not use a fixed list.

**Step 2: Open a new Claude session in a side pane**

Use the Bash tool to run:

```bash
tmux split-window -d -h -t "$(tmux display-message -p '#{pane_id}')" "claude -n '<name>'"
```

This opens a completely fresh Claude session (no shared history with the current pane).

**Step 3: Name the tmux window**

Rename the current tmux window to the chosen name:

```bash
tmux rename-window "<name>"
```

**Step 4: Confirm**

Tell the user:
- A new Claude session named `<name>` is open in the side pane
