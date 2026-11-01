<div align="center">

# Codex Skill for Claude Code

**Get GPT-5.4 second opinions directly from your Claude Code terminal**

<br />

[![Star this repo](https://img.shields.io/github/stars/199-biotechnologies/codex-skill?style=for-the-badge&logo=github&label=%E2%AD%90%20Star%20this%20repo&color=yellow)](https://github.com/199-biotechnologies/codex-skill/stargazers)
&nbsp;&nbsp;
[![Follow @longevityboris](https://img.shields.io/badge/Follow_%40longevityboris-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/longevityboris)

<br />

[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)
&nbsp;
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-blueviolet?style=for-the-badge)](https://docs.anthropic.com/en/docs/claude-code)
&nbsp;
[![GPT-5.4](https://img.shields.io/badge/GPT--5.4-xhigh-orange?style=for-the-badge)](https://openai.com)

---

Cross-check your Claude Code work with OpenAI's GPT-5.4 at maximum reasoning depth. Three modes for different workflows — from quick inline questions to full autonomous code review and implementation.

[Install](#install) | [How It Works](#how-it-works) | [Modes](#modes) | [Prompting Guide](#prompting-codex) | [Contributing](#contributing)

</div>

## Why This Exists

You're deep in a Claude Code session. Something feels off — maybe the architecture, maybe a subtle bug. You want a second pair of eyes, but from a model that thinks differently.

This skill lets you ask GPT-5.4 without leaving your terminal. Claude curates the context, Codex analyzes it with `xhigh` reasoning effort, and the result comes back as a file you can read and act on. No copy-pasting into ChatGPT. No tab-switching.

## Install

```bash
# Copy to your Claude Code skills directory
mkdir -p ~/.claude/skills/codex
curl -sL https://raw.githubusercontent.com/199-biotechnologies/codex-skill/main/SKILL.md \
  -o ~/.claude/skills/codex/SKILL.md
```

**Requirements:**
- [Codex CLI](https://github.com/openai/codex) installed (`npm install -g @openai/codex`)
- ChatGPT subscription (Plus, Pro, Business, or Enterprise) — logged in via `codex login`

## How It Works

```
┌─────────────┐     prompt + context     ┌─────────────┐
│  Claude Code │ ───────────────────────► │  Codex CLI   │
│  (you work   │                          │  GPT-5.4     │
│   here)      │ ◄─────────────────────── │  xhigh       │
└─────────────┘     output .md file      └─────────────┘
```

Claude writes a structured prompt, runs `codex exec` with your subscription auth, and reads the output file back. No API keys needed — it uses your ChatGPT login.

## Modes

| Mode | What Happens | When to Use |
|---|---|---|
| **Quick** | Inline question, direct answer | "What does GPT think about X?" |
| **Analysis** | Claude curates context, Codex analyzes | Architecture review, verification, second opinion |
| **Delegated** | Codex explores the codebase autonomously | Code review, implementation, debugging, testing |

### Quick Mode
```
"Ask codex if this regex handles edge cases"
```
Fast, inline. Codex gets the question, no filesystem access.

### Analysis Mode
```
"Get codex's opinion on our auth architecture"
```
Claude summarizes what it knows into a structured prompt. Codex analyzes without needing to read files — Claude is the context engineer.

### Delegated Mode
```
"Have codex review this repo for security issues"
```
Codex gets full filesystem access and explores on its own. Use `--full-auto` for autonomous work.

## Prompting Codex

Codex is adversarial by nature — it challenges assumptions and proposes alternatives. This is a feature, not a bug. But you need to frame prompts with clear scope and direction.

**The skill includes a built-in prompting guide** that teaches Claude how to:

1. State the goal and direction first (so criticism is constructive)
2. Scope the review explicitly (focus areas + what to leave alone)
3. Pin decisions already made (don't re-litigate framework choices)
4. Specify what kind of feedback you want (bugs? validation? improvements?)

This turns Codex from "confusing contrarian" into "focused expert reviewer."

## Configuration

The skill uses `xhigh` reasoning effort by default — the maximum available for GPT-5.4. It also uses Fast Mode (1.5x faster output via ChatGPT OAuth).

All output is saved to `~/.claude/subagent-results/codex-output-*.md` for persistence across context compaction.

## Trigger Phrases

Say any of these in Claude Code to activate the skill:

- "use codex", "ask codex", "codex check"
- "double check with codex", "verify with codex"
- "codex review", "codex implement"
- "what does GPT think", "OpenAI's take"
- "cross-check with codex"

## Contributing

PRs welcome. If you've found a better prompting pattern or a flag combination that improves output, open a PR.

## License

MIT

---

<div align="center">

Built by [Boris Djordjevic](https://github.com/longevityboris) at [Paperfoot AI](https://paperfoot.si)

<br />

**If this is useful to you:**

[![Star this repo](https://img.shields.io/github/stars/199-biotechnologies/codex-skill?style=for-the-badge&logo=github&label=%E2%AD%90%20Star%20this%20repo&color=yellow)](https://github.com/199-biotechnologies/codex-skill/stargazers)
&nbsp;&nbsp;
[![Follow @longevityboris](https://img.shields.io/badge/Follow_%40longevityboris-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/longevityboris)

</div>
