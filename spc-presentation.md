---
marp: true
theme: uncover
class: invert
paginate: true
style: |
  section {
    background-color: #0d1117;
    color: #e6edf3;
    font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
  }
  h1 {
    color: #58a6ff;
    font-size: 1.8em;
    border-bottom: 2px solid #30363d;
    padding-bottom: 0.3em;
  }
  h2 {
    color: #7ee787;
    font-size: 1.3em;
  }
  code {
    background-color: #161b22;
    color: #f0883e;
    padding: 2px 6px;
    border-radius: 4px;
  }
  pre {
    background-color: #161b22 !important;
    border: 1px solid #30363d;
  }
  table {
    font-size: 0.85em;
  }
  blockquote {
    border-left: 4px solid #58a6ff;
    padding-left: 1em;
    color: #8b949e;
  }
  em {
    color: #8b949e;
  }
---

# Optimize Your Agentic Coding Setup

**For power users who want to squeeze out extra performance.**

*SPC Lunch Demo — Dan Kwon — March 5, 2026*

---

# The Basics

---

## CLAUDE.md: Global vs Project

| Layer | Path | Loaded when? |
|-------|------|-------------|
| **Global** | `~/.claude/CLAUDE.md` | Every session, every project |
| **Project** | `./CLAUDE.md` in project root | In that project only |

Every token in global config costs context in every conversation.

As the file grows, extract to project-level CLAUDE.md and skills.

---

## Example rule: Simplicity Protocol

```markdown
**Core Test**: Can you explain this architecture in 2 minutes?
  NO → simplify. YES → ship it.
**5-Line Rule**: If core logic needs >5 lines of pseudocode,
  it's over-engineered.
**Banned Patterns**: Abstract interfaces for single implementations.
  Config systems for hardcoded values.
  Vendor abstraction until 2+ vendors exist.
**Auto-Reject**: Timeline >2 weeks | Dependencies saving <4 hours
  | Features for <80% users | Abstractions for <3 use cases
**Stop Coding When**: Core workflow works | Happy path smooth
  | Data safe | Basic error handling exists | Tests pass
```

---

## XML priority tags

Not deterministic — a statistical bias on output distribution.

`critical` rules get followed more reliably (but still not guaranteed).

```xml
<rules>
<rule priority="critical">
  Research → Plan → Branch → Code → Test → Document → Merge
</rule>
<rule priority="critical">
  ALWAYS run REAL tests — NEVER use mocks
</rule>
<rule priority="high">
  Stuck >5min? Launch parallel Gemini + Grok analysis
</rule>
</rules>
```

---

## Constant improvement: Auto-memory

`~/.claude/projects/<project>/memory/MEMORY.md`

- Project-level context that loads automatically every session
- `/memory` toggles this on or off (default: ON)
- Force the agent to commit something to memory by asking it to not make the same mistake again

**The config gets better over time because the agent maintains it.**

---

## Post tool call hooks

Three event types: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`

```json
"hooks": {
  "PreToolUse": [{"matcher": "Edit|Write",
    "hooks": [{"type": "command",
      "command": "bash ~/.claude/hooks/detect-secrets.sh"}]}],
  "PostToolUse": [{"matcher": "",
    "hooks": [{"type": "command",
      "command": "bash ~/.claude/hooks/context-usage.sh"}]}],
  "UserPromptSubmit": [{"matcher": "",
    "hooks": [{"type": "command",
      "command": "bash ~/.claude/hooks/register-context.sh"}]}]
}
```

---

## Skills

Invoked by `/skill-name` or automatically when the agent matches the request.

Use skills to store procedures that are useful but don't need to be in CLAUDE.md.

`/audit <scope>` — multi-model audit (133 lines):

```markdown
# Modes
- `/audit <scope>` — Full audit: scan → issues → user answers → execute
- `/audit scan <scope>` — Scan only: produce issues doc, stop

# Rules:
- Consensus = confidence, disagreement = investigate
- "Is this actionable?" is the quality bar
```

---

## Skills vs Custom Agents

| | Skills | Custom Agents |
|--|--------|---------------|
| **Path** | `.claude/skills/` | `~/.claude/agents/` |
| **Invoked by** | `/skill-name` or auto-match | Task tool subagent dispatch |
| **Context** | Runs in main context | **Isolated context** |

**Context isolation is the key difference.** Agents don't pollute the main conversation.

---

## Custom Agents

`~/.claude/agents/` — available to Task tool as subagent types.

```markdown
---
name: code-reviewer
description: Expert code review specialist.
tools: Read, Grep, Glob, Bash
---
When invoked:
1. Run `git diff` to see recent changes
2. Focus on modified files
3. Begin review immediately
Feedback: Critical | Warning | Suggestion
```

Use when you need a skill but don't want to add to the main agent context.

---

# Interesting Concepts

---

## Context window management is everything

High context window usage (even without reaching limits) degrades reasoning.

> *"Context Length Alone Hurts LLM Performance Despite Perfect Retrieval"*

Compaction is catastrophic. When the 200k window fills, it auto-summarizes. Reasoning quality collapses.

Use `/clear` when old conversation context is not needed.

---

## Persona model: garbage in, garbage out

Input token "intelligence" affects output token "intelligence."

- Paper: Small prompt variations make a big difference for Llama
- Anthropic's **Persona Selection Model**: LLMs simulate diverse "personas" during pretraining. Post-training refines personas.
- Prompts that invoke "intelligent" personas produce more "intelligent" output

Maybe: Does prompt engineering matter more for Gemini than Opus?

---

## Models are different, not better

Using the same model for multi-agent debate may have consistent blind spots.

| Model | Strength | Blind spot |
|-------|----------|-----------|
| **Claude Opus** | Avoids hallucinations | Often "lazy" |
| **Grok** | Doggedly looks for disconfirming info | Risk of hallucination |
| **Gemini** | Extremely powerful | Lacks common sense |
| **Codex** | Fixing thorny bugs | Not suited for analysis |

When models agree → corroboration. When they disagree → investigate.

---

# The Techniques

---

## Custom "plan mode" via CLAUDE.md

```markdown
<rule priority="critical">
  Research → Plan → Branch → Code → Test → Document → Merge
</rule>
```

```markdown
1. **Research**: Use Task tool for 2+ aspects before coding
2. **Plan**: Create PRD for large tasks; decompose for parallel work
3. **Branch**: Create feature branch before code changes
4. **Code**: Implement with simplicity principles
5. **Test**: Run REAL tests locally
6. **Document**: Update relevant docs
7. **Merge**: Commit with "why" not "what"
```

---

## Multi-agent research and planning

```markdown
<rule priority="critical">When launching subagents, use the Task Tool,
Codex CLI, Grok API, and Gemini CLI.</rule>
```

CLI invocations:

| Model | Command |
|-------|---------|
| Codex | `codex -m gpt-5.3-codex exec --full-auto "task"` |
| Gemini | `gemini -m gemini-3.1-pro-preview -p "prompt"` |
| Grok | Python + curl (no CLI — see next slide) |

---

## Grok invocation (API via curl)

```bash
python3 << 'EOF'
import json
code = open('file.py').read()
req = {"messages": [{"role": "user",
  "content": f"Review:\n{code}"}],
  "model": "grok-4-1-fast-reasoning", "temperature": 0}
json.dump(req, open('/tmp/req.json', 'w'))
EOF
curl -X POST https://api.x.ai/v1/chat/completions \
  -H "Authorization: Bearer $GROK_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/req.json
```

---

## Tone = intelligence: the register-context hook

Next-token prediction: there is no separation between tone and intelligence. A junior-engineer-sounding token IS a junior-engineer-intelligence token.

A hook fires on every prompt and injects:

> *"This conversation operates at principal-engineer depth. Lead with mechanisms and tradeoffs, not summaries."*

**Every response is biased toward higher-register output automatically.**

---

## Context discipline: make the invisible visible

A hook injects a fuel gauge after every action:

> `85k/200k (42%)`

```bash
TOTAL=$(echo "$USAGE" | jq '.message.usage |
  (.input_tokens + .cache_creation_input_tokens
  + .cache_read_input_tokens)')
K=$(( TOTAL / 1000 ))
PCT=$(( TOTAL * 100 / 200000 ))
```

**At 60%:** Evaluate if you can finish. Write remaining tasks to disk.
**At 80%:** Stop new work. Wrap up and end session.

---

## "Why not just use Cursor?"

Cursor, Copilot Workspace, Aider, Replit Agent — all good tools.

What they don't give you:

- **Auto-memory** — self-improving config across sessions
- **Multi-model orchestration** from a single session
- **Constraint architecture** (coming next)

If you want zero-config AI coding → use Cursor.
If you want a system that compounds → build config.

---

## Defense in depth

| Layer | What it blocks | When |
|-------|---------------|------|
| **PreToolUse hook** | AWS keys, GitHub tokens, Stripe keys | Before the file is written |
| **Git pre-commit** | Same patterns on staged changes | Before the commit |
| **Deny rules** | `git push --force`, `git reset --hard` | Before the command runs |

---

# The Counterintuitive Lesson

---

## Why constraints beat capabilities

I spent two weeks building interview analysis instead of studying. The output was excellent. The **direction** was wrong.

The fix wasn't a better model. It was **constraints.**

**The tool that makes you 10x faster at the wrong thing is not a 10x tool.**

---

## One line that changed everything

```xml
<rule priority="critical">
  Action terminates analysis. If a decision is clear,
  stop analyzing and execute.
</rule>
```

The agent enforces it on itself.

When I start spiraling into another analysis document, it stops and says: *"you've already decided — do the thing."*

It works because the constraint is in the config, not in my willpower.

---

## More constraints that changed behavior

| Constraint | What it stopped |
|------------|----------------|
| "No motivational fluff. No 'you've got this!'" | Cheerleading instead of honest assessments |
| "Tables > prose. Bullets > paragraphs." | Verbose output (cut ~40%) |
| "Search existing files before creating new ones." | File sprawl — duplicate strategy docs everywhere |

None of these are capabilities. They're guardrails.

---

# Quick Start — do these today

1. **Install the context-usage hook.** Give the LLM a fuel gauge.
2. **Create CLAUDE.md.** Five rules. Under 100 lines.
3. **Use the Task tool for research.** Parallel subagents preserve context.
4. **Track agent errors in memory.** Error rate drops session over session.
5. **Add constraint rules.** Start with one: *"If the decision is clear, stop analyzing and execute."*

---

# Practical Notes

**Cost:** ~$200/mo total (Claude Max + negligible API costs for Grok/Gemini/Codex)

**Setup:** `setup.sh` takes ~10 minutes. Building CLAUDE.md rules takes weeks of iteration — that's the real investment.

**Repos:**
- Slide deck: github.com/pahdo/spc-dotfiles-demo
- Full notes: github.com/pahdo/agentic-coding-notes

---

## What else I've been using

- **/remote-control** — note: `/clear` doesn't work in remote sessions
- **Tailscale + Screens** (iOS) — start new remote-control sessions from phone
- **/sandbox** — like dangerously-skip-permissions, but safer
- **Obsidian** — workbook management, synced via Obsidian Sync

---

## Reference

> **Boris Cherny** (@bcherny)
>
> Invest in your CLAUDE.md. After every correction, end with:
> "Update your CLAUDE.md so you don't make that mistake again."
>
> Claude is eerily good at writing rules for itself.

---

### Thanks for reading! Please feel free to suggest any improvements
