---
name: fork-pane
description: Fork the current Claude session and open the fork in a new tmux side pane — one command, no extra steps
allowed-tools:
  - Bash
---

Fork this Claude session and open the fork in a new tmux side pane.

**Step 1: Get the session ID**

If `$ARGUMENTS` is provided and looks like a UUID, use it as the session ID.

Otherwise, use the Bash tool to find the current session ID (most recently modified entry in session-env):

```bash
ls -t ~/.claude/session-env/ | head -1
```

**Step 2: Open a forked session in a new tmux pane**

Use the Bash tool to run:

```bash
tmux split-window -h "claude -r <session-id> --fork-session"
```

The `--fork-session` flag resumes the current session AND creates a new independent branch of it. This means both panes start from the same conversation state and can diverge independently.

**Step 3: Confirm**

Tell the user:
- The fork is now open in the right pane (new independent branch)
- The current pane is the original, unchanged
- Both start from the same conversation state
- How to switch: `Ctrl+b →` / `Ctrl+b ←`
