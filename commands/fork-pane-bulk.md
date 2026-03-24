---
name: fork-pane-bulk
description: Fork the current Claude session N times, opening each fork in its own tmux side pane with a randomly generated name
allowed-tools:
  - Bash
---

Fork this Claude session N times, each in a new tmux side pane within the current window.

**Step 1: Parse arguments**

`$ARGUMENTS` has the form: `<count> [<session-id>]`

- The first token is the **count** (required) — how many forks to create
- The second token, if it looks like a UUID, is the **session ID** to fork from

If no session ID is found in `$ARGUMENTS`, find it with the Bash tool:

```bash
ls -t ~/.claude/session-env/ | head -1
```

**Step 2: Generate names**

Generate `<count>` unique random human-readable names in the style of `<adjective>-<noun>` (e.g. `swift-canyon`, `amber-tide`, `hollow-pine`). All names must be distinct. Pick words at random — do not use a fixed list.

**Step 3: Create all forks in one Bash call**

Use the Bash tool to run ONE single command with all the names and the session ID substituted in — do not loop across multiple Bash calls:

For 3 forks named `swift-canyon`, `amber-tide`, `hollow-pine` with session ID `<session-id>`, the command looks like:

```bash
CLAUDE=$(which claude) && \
SESS=$(tmux display-message -p '#{session_name}') && \
WIND=$(tmux display-message -p '#{window_index}') && \
for NAME in "swift-canyon" "amber-tide" "hollow-pine"; do \
  tmux split-window -d -h -t "${SESS}:${WIND}" "$CLAUDE -r <session-id> --fork-session -n '${NAME}'"; \
done && \
tmux select-layout -t "${SESS}:${WIND}" tiled && \
tmux set-option -g mouse on
```

Substitute the real names and real session ID before running. Each pane runs Claude directly. The `--fork-session` flag resumes the session AND creates a new independent branch. Every fork starts from the same conversation state and diverges independently. The `select-layout tiled` at the end distributes all panes evenly.

**Step 4: Confirm**

Tell the user:
- How many forks were created
- The list of pane names (one per line)
