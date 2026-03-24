---
name: new-pane-bulk
description: Open N fresh Claude sessions, each in its own tmux side pane with an auto-generated name
allowed-tools:
  - Bash
---

Open N brand-new Claude sessions, each in its own tmux side pane within the current window.

**Step 1: Parse arguments**

`$ARGUMENTS` has the form: `<count> [<name1> <name2> ...]`

- The first token is the **count** (required) — how many new sessions to create
- Any remaining tokens are treated as explicit **pane names**, assigned in order
- If fewer names are provided than `<count>`, generate random human-readable names for the remainder in the style of `<adjective>-<noun>` (e.g. `silver-drift`, `pale-torch`, `keen-vale`). All names must be distinct. Pick words at random — do not use a fixed list.

**Step 2: Create all sessions in one Bash call**

Use the Bash tool to run ONE single command with all the names substituted in — do not loop across multiple Bash calls:

For 3 sessions named `silver-drift`, `pale-torch`, `keen-vale`, the command looks like:

```bash
CLAUDE=$(which claude) && \
SESS=$(tmux display-message -p '#{session_name}') && \
WIND=$(tmux display-message -p '#{window_index}') && \
for NAME in "silver-drift" "pale-torch" "keen-vale"; do \
  tmux split-window -d -h -t "${SESS}:${WIND}" "$CLAUDE -n '${NAME}'"; \
done && \
tmux select-layout -t "${SESS}:${WIND}" tiled
```

Substitute the real names before running. Each pane runs Claude directly. The `select-layout tiled` at the end distributes all panes evenly.

**Step 3: Confirm**

Tell the user:
- How many sessions were created
- The list of pane names (one per line)
