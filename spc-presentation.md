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

# Agentic Programming Workflow

**The highest-leverage AI configs are constraints, not capabilities.**

*SPC Lunch Demo — Dan Kwon*

---

# Two failure modes nobody warns you about

---

## 1. Your AI forgets mid-conversation

Claude Code has a 200k-token context window. When it fills, it auto-summarizes — **compaction.**

Compaction is catastrophic. Reasoning quality collapses. There is no good mental model for what survives.

Even without compaction, longer context = worse reasoning.

> Paper: *"Context Length Alone Hurts LLM Performance Despite Perfect Retrieval"*

Think of it as **working memory.** Drop a 500-page manual on a junior employee's desk — they'll forget the important parts. Compaction is the AI quietly throwing away your instructions.

---

## 2. Your AI helps you do the wrong work at lightspeed

I built 20+ files of interview analysis — strategy documents, company research, competitive frameworks — when I should have been drilling Python basics.

The agent never pushed back. The output was excellent. The **direction** was wrong.

An AI agent will help you do the wrong work with extraordinary thoroughness.

---

# The System

---

## The Employee Handbook: config architecture

Everything lives in a dotfiles repo, symlinked to `~/.claude/`. One `setup.sh` bootstraps it.

| Layer | Analogy | Loaded when? |
|-------|---------|-------------|
| **Global rules** | Company handbook | Every session |
| **Project CLAUDE.md** | Team SOPs | In that project |
| **Skills** | Reference manual | On-demand only |

Every token in global config costs context in every conversation.

*My first CLAUDE.md: ~320 lines. Now: under 100.*

---

## The Boris Cherny loop

> "After every correction, end with: Update your CLAUDE.md so you don't make that mistake again."

I corrected an Obsidian formatting issue twice. Told the agent to write a rule.

Third session onward — no repeats.

**The config gets better over time because the agent maintains it.**

---

## XML priority tags

Not deterministic — a statistical bias on output distribution.

`critical` rules get followed more reliably than untagged rules.

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

## Tone = intelligence: the register-context hook

A hook fires on every prompt and injects:

> *"This conversation operates at principal-engineer depth. Lead with mechanisms and tradeoffs, not summaries."*

Next-token prediction: there is no separation between tone and intelligence. A junior-engineer-sounding token IS a junior-engineer-intelligence token.

**The hook biases every response toward higher-register output without requiring formal prompts.**

---

## Context discipline: make the invisible visible

Three defenses against the working-memory problem:

**1. Keep config small.** Under 100 lines.

**2. Make context visible.** A hook injects a fuel gauge after every action:

> `85k/200k (42%)`

At 60%: evaluate if you can finish. At 80%: stop and document state.

**3. Delegate to subagents.** Task tool results return summarized — 50-step research = a few paragraphs, not 50 tool calls.

---

## 🔴 DEMO 1: The Context Fuel Gauge

Run a Claude Code command → point to the context percentage.

*"Instead of guessing when the AI is about to lose its memory, it now sees its own capacity and stops itself."*

---

## Cross-functional review: multi-model orchestration

| Model | Role | Analogy |
|-------|------|---------|
| **Claude Opus 4.6** | Deep analysis, writing | Your meticulous lead |
| **Codex gpt-5.3** | Long autonomous coding | Your workhorse engineer |
| **Grok 4.1** | Code review, contrarian | Your auditor who disagrees |
| **Gemini 3.1 Pro** | Architecture, security | Your second opinion |

---

## Why not just one model?

From a four-model audit on my own documents:

| Model | Strength | Blind spot |
|-------|----------|-----------|
| **Claude** | Fact-checking, nuance | Accepts others' citations uncritically |
| **Grok** | Contrarian stress-tests | Fabricates legal citations |
| **Gemini** | Red-teaming, frameworks | Weaker on deep ambiguity |
| **Codex** | Long autonomous coding | Not suited for analysis |

**Example:** Grok cited a real IRS ruling for the wrong topic. Claude accepted it. Gemini caught it.

When models agree → corroboration.
When they disagree → investigate.

---

## 🔴 DEMO 2: Multi-Model Audit

Launch a multi-model audit → show agents fanning out in parallel → show where they disagree.

*"If you ask one person for advice, you get a blind spot. If you ask three different models trained on different data, you find the truth."*

---

## "Why not just use Cursor?"

Cursor, Copilot Workspace, Aider, Replit Agent — all good tools.

What they don't give you:

- **Boris Cherny loop** — self-improving config
- **Multi-model orchestration** from a single session
- **Constraint architecture** (coming next)

If you want zero-config AI coding → use Cursor.
If you want a system that compounds → build config.

---

## Defense in depth

| Layer | What it blocks | When |
|-------|---------------|------|
| **PreToolUse hook** | AWS keys, GitHub tokens, Stripe keys, private keys | Before the file is written |
| **Git pre-commit** | Same patterns on staged changes | Before the commit |
| **Deny rules** | `git push --force`, `git reset --hard`, etc. | Before the command runs |

*Known gaps documented. Belt and suspenders.*

---

## Observability: the audit trail

Every skill/task invocation gets logged:

```
2026-02-28 14:32 | skill | interview-prep | claude-life
2026-02-28 14:35 | task  | Explore        | openclaw
2026-02-28 15:01 | skill | audit          | claude-life
```

This is how I know interview-prep is heavy and env-setup is never used. **Data, not guesses.**

---

## Self-correcting memory

Claude is stateless. Every session starts from zero.

Fake persistent learning with a correction file (`memory/MEMORY.md`) that loads every conversation:

```markdown
## Hallucination Prevention
- Never present agent assessments as company feedback
- Comp figures: note source (company, posting, or estimate)

## Tool Gotchas
- Edit: use smaller unique target strings
- Grep: sometimes ENOENT — use Read as fallback
```

**Not machine learning. A manually curated correction file. The feedback loop lives in config, not in the model.**

---

# The Counterintuitive Lesson

---

## Skills codify what you learned

`/interview-prep` — 102 lines, 15+ transcripts, 5 companies.

I tracked what I actually used in a 64-minute panel interview:

- Scripted Q&A answers: **used 3 out of 16**
- Ad-hoc scaffolding: **used constantly**

Rule: don't write scripts, build frameworks the agent fills in real-time.

*"Mechanical fixes over personality advice. 'Count to 3 after finishing. Sip water.' NOT 'Be more confident.'"*

---

## Escape velocity: why constraints beat capabilities

The changes that most improved my output weren't new tools, new models, or new skills.

They were **constraints.**

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

**"No motivational fluff. No 'you've got this!'"**
→ Stopped cheerleading, started honest assessments.

**"Tables > prose. Bullets > paragraphs."**
→ Cut verbose output by ~40%.

**"Search existing files before creating new ones."**
→ Stopped file sprawl. I had duplicate strategy docs everywhere.

---

## The takeaway

When you get a powerful AI tool, your instinct is to give it more capabilities.

The higher-leverage move is to give it **constraints.**

**The tool that makes you 10x faster at the wrong thing is not a 10x tool.**

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

**Repo:** github.com/pahdo/spc-dotfiles-demo

---

## Reference

> **Boris Cherny** (@bcherny)
>
> Invest in your CLAUDE.md. After every correction, end with:
> "Update your CLAUDE.md so you don't make that mistake again."
>
> Claude is eerily good at writing rules for itself.
