# tmux-kit

A Claude Code plugin for managing multiple Claude sessions across tmux panes — fork sessions, spawn fresh ones, and merge their context back together.

**Requirements:** Claude Code CLI, [tmux](https://github.com/tmux/tmux)

## Installation

### Step 1 — Start a tmux session

> ${\color{red}\textbf{Claude\ must\ be\ running\ inside\ tmux.\ Run\ this\ first:}}$

```bash
tmux
```

Or with a named session:

```bash
tmux new -s my-session
```

### Step 2 — Launch Claude Code

```bash
claude
```

### Step 3 — Install tmux-kit

#### As a plugin (recommended)

```
/plugin marketplace add abhinavbajpai2012/tmux-kit
/plugin install tmux-kit
```

Commands are then available namespaced:

```
/tmux-kit:fork-pane
/tmux-kit:new-pane
/tmux-kit:merge-pane
...
```

#### Manual installation

Copy any command file from `commands/` into your `~/.claude/commands/` directory:

```bash
cp commands/<command-name>.md ~/.claude/commands/
```

Commands are available immediately in any Claude session without a restart, using their short name (e.g. `/fork-pane`).

---

## Commands

### `/fork-pane`

Fork the current Claude session and open the fork in a new tmux side pane. Both panes start from the same conversation state and diverge independently. The pane and Claude session are given a matching auto-generated name (or one you provide).

**Usage:**
```
/fork-pane                        # auto-generated name
/fork-pane bright-canyon          # named fork
/fork-pane <session-id>           # fork a specific session (auto-named)
/fork-pane <session-id> my-name   # fork a specific session with a name
```

---

### `/fork-pane-bulk`

Fork the current Claude session N times, each in its own named tmux window.

**Usage:**
```
/fork-pane-bulk 3                     # 3 auto-named forks
/fork-pane-bulk 3 <session-id>        # 3 forks of a specific session
```

---

### `/new-pane`

Open a brand-new Claude session (no shared history) in a tmux side pane.

**Usage:**
```
/new-pane                  # auto-generated name
/new-pane my-experiment    # named session
```

---

### `/new-pane-bulk`

Open N fresh Claude sessions, each in its own named tmux window.

**Usage:**
```
/new-pane-bulk 3               # 3 auto-named sessions
/new-pane-bulk 3 alpha beta    # 2 explicit names, 1 auto-generated
```

---

### `/merge-pane`

Merge the conversation context from a named Claude session into the current session. Finds the session by its tmux window name, reads its conversation history, and synthesizes a context summary directly into the current conversation.

**Usage:**
```
/merge-pane bright-canyon
```

---

### `/merge-pane-bulk`

Merge context from multiple named sessions at once. Produces per-session summaries and a cross-session observations section.

**Usage:**
```
/merge-pane-bulk bright-canyon alpha my-experiment
```
