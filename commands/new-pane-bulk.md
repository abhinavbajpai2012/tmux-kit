---
name: new-pane-bulk
description: Open N fresh Claude sessions, each in its own tmux window with an auto-generated name
allowed-tools:
  - Bash
---

Open N brand-new Claude sessions, each in its own tmux window with a unique auto-generated name.

**Step 1: Parse arguments**

`$ARGUMENTS` has the form: `<count> [<name1> <name2> ...]`

- The first token is the **count** (required) — how many new sessions to create
- Any remaining tokens are treated as explicit **window names**, assigned in order
- If fewer names are provided than `<count>`, generate random human-readable names for the remainder in the style of `<adjective>-<noun>` (e.g. `silver-drift`, `pale-torch`, `keen-vale`). All names must be distinct. Pick words at random — do not use a fixed list.

**Step 2: Create each new session**

For each session (repeat `<count>` times), use the Bash tool to run this single command (do NOT split into multiple calls):

```bash
SESS=$(tmux display-message -p '#{session_name}') && tmux new-window -d -n "<name>" -t "${SESS}:" && tmux split-window -d -h -t "${SESS}:<name>" "claude -n '<name>'"
```

Capturing `SESS` inline ensures the session name is always fresh and never carried incorrectly across separate tool calls. The `-d` flags create everything in the background without stealing focus.

Run each session as a separate Bash call so failures are isolated.

**Step 4: Confirm**

Tell the user:
- How many new sessions were created
- The list of window names (one per line)
- Each is a fresh, independent Claude session
- How to navigate: `Ctrl+b w` to see all windows, `Ctrl+b '` to jump by name, or `tmux select-window -t <name>`
