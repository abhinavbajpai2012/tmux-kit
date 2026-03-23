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
SESS=$(tmux display-message -p '#{session_name}') && \
for NAME in "swift-canyon" "amber-tide" "hollow-pine"; do \
  tmux new-window -d -n "$NAME" -t "${SESS}:" && \
  tmux split-window -d -h -t "${SESS}:${NAME}" "claude -r <session-id> --fork-session -n '${NAME}'"; \
done
```

Substitute the real names and real session ID before running. The `--fork-session` flag resumes the session AND creates a new independent branch. Every fork starts from the same conversation state and diverges independently.

**Step 4: Confirm**

Tell the user:
- How many forks were created
- The list of window names (one per line)
- Each fork is an independent branch starting from the same conversation state
- How to navigate: `Ctrl+b w` to see all windows, `Ctrl+b '` to jump by name, or `tmux select-window -t <name>`
