# claude-tools

A collection of custom Claude Code slash commands.

## Installation

Copy any command file from `commands/` into your `~/.claude/commands/` directory:

```bash
mkdir -p ~/.claude/commands
cp commands/<command-name>.md ~/.claude/commands/
```

The command is instantly available in any Claude session — no restart needed.

## Commands

### `/fork-pane`

Fork your current Claude session and open the fork in a new tmux side pane — all in one command.

**Requirements:** Claude Code CLI, tmux

**Usage:** Inside a Claude session running inside tmux, run:
```
/fork-pane
```

Optionally pass a specific session ID to fork:
```
/fork-pane <session-id>
```

A new pane opens on the right with an independent fork of your conversation. Both branches diverge from that point.

Switch panes: `Ctrl+b →` / `Ctrl+b ←`
