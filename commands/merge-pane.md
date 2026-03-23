---
name: merge-pane
description: Merge the conversation context from a named tmux/Claude session into the current session
allowed-tools:
  - Bash
---

Merge the conversation context from a named Claude session into the current session.

**Step 1: Parse arguments**

`$ARGUMENTS` has the form: `<name>`

- `<name>` is the tmux window name / Claude session display name (required)

**Step 2: Find the session ID**

First, try to find the session while it is still running by looking up its PID via tmux:

```bash
tmux list-panes -t "<name>" -F "#{pane_pid}" 2>/dev/null
```

If a PID is found, find its child processes (the actual claude process):

```bash
pgrep -P <pane_pid>
```

Then find the matching session file in `~/.claude/sessions/`:

```bash
for f in ~/.claude/sessions/*.json; do
  python3 -c "
import json, sys
d = json.load(open('$f'))
if d.get('pid') == <pid>:
    print(d['sessionId'])
" 2>/dev/null
done
```

If the session is no longer running (no tmux window or no matching PID), fall back to searching by slug across all project JSONL files:

```bash
python3 -c "
import json, glob, os, sys
name = '<name>'
files = sorted(glob.glob(os.path.expanduser('~/.claude/projects/**/*.jsonl'), recursive=True), key=os.path.getmtime, reverse=True)
for f in files:
    with open(f) as fh:
        for line in fh:
            try:
                d = json.loads(line)
                if d.get('slug') == name and d.get('sessionId'):
                    print(d['sessionId'], f)
                    sys.exit(0)
            except:
                pass
"
```

**Step 3: Read the session's conversation**

Once you have the session ID and JSONL file path, extract all `user` and `assistant` message entries:

```bash
python3 -c "
import json, os, glob

session_id = '<session-id>'

# Find the JSONL file for this session
files = glob.glob(os.path.expanduser('~/.claude/projects/**/*.jsonl'), recursive=True)
target = None
for f in files:
    if session_id in f:
        target = f
        break

if not target:
    # Search by sessionId field inside files
    for f in files:
        with open(f) as fh:
            line = fh.readline()
            try:
                d = json.loads(line)
                if d.get('sessionId') == session_id:
                    target = f
                    break
            except:
                pass

if not target:
    print('Session file not found')
    exit(1)

turns = []
with open(target) as fh:
    for line in fh:
        try:
            d = json.loads(line)
            if d.get('type') == 'user':
                msg = d.get('message', {})
                content = msg.get('content', '')
                if isinstance(content, list):
                    text = ' '.join(c.get('text','') for c in content if c.get('type') == 'text')
                else:
                    text = str(content)
                if text.strip():
                    turns.append(('User', text.strip()[:500]))
            elif d.get('type') == 'assistant':
                msg = d.get('message', {})
                content = msg.get('content', [])
                text = ' '.join(c.get('text','') for c in content if isinstance(c, dict) and c.get('type') == 'text')
                if text.strip():
                    turns.append(('Assistant', text.strip()[:500]))
        except:
            pass

for role, text in turns:
    print(f'[{role}]: {text}')
    print()
"
```

**Step 4: Synthesize and inject context**

Read the extracted conversation turns and synthesize a concise context summary covering:
- The main goal or problem being worked on in that session
- Key decisions made
- Work completed (files changed, features built, bugs fixed)
- Relevant conclusions or next steps that were identified

Present this synthesis directly in the current conversation as injected context, clearly framed as:

> **Merged context from session "<name>":**
> [your synthesis here]

Then tell the user:
- Context from `<name>` has been merged into the current session
- Claude now has awareness of what was done in that session
- Any relevant follow-up they might want to take
