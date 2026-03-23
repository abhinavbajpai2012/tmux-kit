---
name: merge-pane-bulk
description: Merge conversation context from multiple named tmux/Claude sessions into the current session
allowed-tools:
  - Bash
---

Merge the conversation context from multiple named Claude sessions into the current session.

**Step 1: Parse arguments**

`$ARGUMENTS` has the form: `<name1> [<name2> ...]`

- One or more session names (space-separated), each being a tmux window name / Claude session display name
- At least one name is required

**Step 2: For each name, find the session ID**

Repeat the following for every name in the list.

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
import json
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

Collect all resolved session IDs (note any names that could not be resolved and warn the user).

**Step 3: Read each session's conversation**

For each resolved session ID, extract all `user` and `assistant` message entries using the same JSONL lookup as above:

```bash
python3 -c "
import json, os, glob

session_id = '<session-id>'

files = glob.glob(os.path.expanduser('~/.claude/projects/**/*.jsonl'), recursive=True)
target = None
for f in files:
    if session_id in f:
        target = f
        break

if not target:
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

Run lookups sequentially, one session at a time.

**Step 4: Synthesize and inject all contexts**

For each session, synthesize a concise context summary covering:
- The main goal or problem being worked on
- Key decisions made
- Work completed (files changed, features built, bugs fixed)
- Relevant conclusions or next steps identified

Present all summaries together in the current conversation, clearly framed:

> **Merged context from N sessions:**
>
> **[name1]:** [synthesis]
>
> **[name2]:** [synthesis]
>
> ...

Then, if there are common themes, decisions, or conflicts across the sessions, add a brief **Cross-session observations** section highlighting them.

Finally, tell the user:
- How many sessions were successfully merged
- Any names that could not be resolved (and why)
- Claude now has awareness of what was done across all those sessions
