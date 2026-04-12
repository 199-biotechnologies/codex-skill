---
name: codex
description: >-
  Use OpenAI Codex CLI (GPT-5.4) for second opinions, analysis, verification,
  code review, or delegated autonomous tasks. Triggers when user says "use
  codex", "ask codex", "codex check", "double check with codex", "verify with
  codex", "codex analyze", "codex review", "codex implement", "have codex do",
  "cross-check with codex", "what does GPT think", "OpenAI's take", or wants
  an alternative AI perspective. See also: the Gemini skill for an alternative
  external AI. When the user doesn't specify which, prefer Codex for
  code-heavy tasks and Gemini for long-context analysis.
---

# Codex CLI - GPT-5.4

Second opinion, verification, analysis, code review, or autonomous task delegation using OpenAI's Codex CLI.

## Defaults

**Model: `gpt-5.4`** — always. **Reasoning: `xhigh`** — always, unless the user explicitly requests a different effort level.

ALWAYS pass them explicitly in commands — never rely on config.toml:
```
-m gpt-5.4 -c model_reasoning_effort="xhigh"
```

Override only when the user says so (e.g., "use low reasoning" → `-c model_reasoning_effort="low"`).

## Fast Mode

`fast_mode` is a stable feature flag (enabled by default). Uses `service_tier = "fast"` under the hood — same gpt-5.4 model, ~1.5x faster output, 2x credit cost. **Only works with ChatGPT OAuth login** (not API key).

- Already active by default. No extra flags needed.
- Disable per invocation: `--disable fast_mode`
- Re-enable: `--enable fast_mode`
- Interactive toggle: `/fast`

## Models

| Model | Notes |
|---|---|
| `gpt-5.4` | **Default. Always use this.** Requires ChatGPT OAuth login (`codex login`). |
| `gpt-5.3-codex` | Fallback ONLY if gpt-5.4 fails. Works with both OAuth and API key. |
| `gpt-5.2-codex` | Older fallback. Works with both OAuth and API key. |
| `gpt-5.1-codex-max` | Multi-context-window. Good for project-scale refactors. |
| `gpt-5.1-codex-mini` | Cost-effective (~4x more usage). |

## Prompting Codex: Critical Guidance

**Codex is adversarial and critical by default.** It will challenge assumptions, propose alternatives, and sometimes argue the opposite direction. This is its strength — but only if you frame the prompt correctly. Without clear scope and direction, Codex becomes confusing rather than helpful.

**Before prompting Codex, ask yourself:** Is the user's intent clear enough to write a focused prompt? If not, ask the user to clarify before invoking Codex.

### Prompt Framing Rules

1. **State the goal and direction first.** Tell Codex what we're trying to achieve and WHY, so its criticism is constructive within that frame:
   - BAD: "Review this authentication code"
   - GOOD: "We're implementing OAuth2 PKCE for a mobile app. Review this auth code for security gaps and spec compliance — we've committed to this approach."

2. **Scope the review explicitly.** Tell Codex what to focus on AND what to leave alone:
   - BAD: "What do you think of this architecture?"
   - GOOD: "Review the data layer architecture. Focus on: query performance, connection pooling, and error recovery. Don't suggest switching ORMs — we're committed to Prisma."

3. **State what kind of feedback you want:**
   - "Find bugs and security issues" → Codex hunts for problems
   - "Verify this approach is sound" → Codex validates or challenges the reasoning
   - "Suggest improvements within our current approach" → Codex stays constructive
   - "What are we missing?" → Codex looks for blind spots

4. **Provide the spirit/direction of the work.** Codex needs to understand whether something is a prototype, production code, a learning exercise, or a performance-critical hot path. This changes what matters.

5. **Pin decisions that are already made.** If the user has chosen a framework, pattern, or approach, tell Codex: "We are using X. Don't suggest replacing X — focus on making X work well."

### Prompt Template (for Analysis/Delegated modes)

```markdown
## Goal
[What we're building/fixing and WHY — the spirit of the work]

## Decisions Already Made
[Frameworks, patterns, approaches that are locked in — don't re-litigate these]

## Context
[Code, architecture, relevant snippets]

## What We Need From You
[Specific, scoped questions — numbered]

## Out of Scope
[What NOT to comment on]
```

## Mode Selection

Choose mode based on the **intent** — is Claude providing context, or is Codex doing its own work?

---

### 1. Quick Mode — Simple questions, short answers

For questions where Claude already has the answer context and just needs a second opinion.

**Bash tool timeout: 120000ms**

```bash
codex exec -m gpt-5.4 --skip-git-repo-check --sandbox read-only --ephemeral \
  -c model_reasoning_effort="xhigh" "<PROMPT>"
```

---

### 2. Analysis Mode — Claude curates context, Codex analyzes

When Claude has already read the code/docs and can provide a focused brief. Use this when Codex does NOT need filesystem access — Claude is the context engineer.

**Bash tool timeout: 300000ms**

**Step 1:** Write a structured prompt file using the Write tool:
```bash
PROMPT_FILE="~/.claude/subagent-results/codex-prompt-$(date +%s).md"
OUTPUT_FILE="~/.claude/subagent-results/codex-output-$(date +%s).md"
```

Use the Prompt Template above. Always include Goal, Decisions Already Made, and What We Need From You.

**Step 2:** Invoke with file I/O:
```bash
codex exec -m gpt-5.4 \
  --sandbox read-only \
  --ephemeral \
  -c model_reasoning_effort="xhigh" \
  -o "$OUTPUT_FILE" \
  - < "$PROMPT_FILE"
```

**Step 3:** Read output file with the Read tool and present to user.

---

### 3. Delegated Mode — Codex works autonomously

When Codex needs to explore a codebase, review code, write files, run commands, or do independent work. This gives Codex full filesystem access and tool use. Use when:
- User wants Codex to **review** a codebase itself
- User wants Codex to **explore** and form its own opinion
- User wants Codex to **implement** something
- User wants Codex to **run tests**, **debug**, or **investigate**

**Do NOT use `--ephemeral` in Delegated Mode** — sessions may need persistence for multi-step work.

**Bash tool timeout: 600000ms. For tasks likely to exceed 10 minutes, use `run_in_background: true` on the Bash tool and check the output file later.**

```bash
OUTPUT_FILE="~/.claude/subagent-results/codex-output-$(date +%s).md"
codex exec -m gpt-5.4 \
  -C /path/to/project \
  --full-auto \
  -c model_reasoning_effort="xhigh" \
  -o "$OUTPUT_FILE" \
  "<PROMPT>"
```

For **code review** specifically, use the built-in review subcommand:
```bash
codex exec review -m gpt-5.4 -C /path/to/project --full-auto \
  -c model_reasoning_effort="xhigh" -o "$OUTPUT_FILE"
```

Or the top-level review command:
```bash
codex review -m gpt-5.4 -C /path/to/project \
  -c model_reasoning_effort="xhigh" -o "$OUTPUT_FILE"
```

**No output constraints in delegated mode.** Let Codex produce whatever it needs. The `-o` flag captures only the final message, so context stays manageable.

---

## Choosing the Right Mode

| Signal | Mode | Why |
|---|---|---|
| "Ask codex about X" + Claude already has context | **Analysis** | Claude curates, Codex analyzes. Efficient. |
| "Have codex review the repo" | **Delegated** | Codex needs to read files itself. |
| "Quick question for codex" | **Quick** | Inline, fast, minimal. |
| "Get codex's opinion on this architecture" + code already explored | **Analysis** | Claude summarizes what it knows. |
| "Have codex explore and analyze this project" | **Delegated** | Codex does its own exploration. |
| "Use codex to implement X" | **Delegated** | Codex needs write access. |

## Output Handling

**ALWAYS use `-o <file>` for output** (except Quick mode where inline is fine). This captures only the final response, stripping session headers, tool-use traces, and thinking. Read it with the Read tool.

- For **Analysis Mode**: Summarize if presenting inline, or give full output if user wants detail.
- For **Delegated Mode**: Present the full output — Codex may have done substantial work. Check if it modified files.
- **ALWAYS verify output exists**: if the output file is empty or missing, check stderr and report.

## Error Handling

- If the command exits non-zero, check stderr (never suppress it with `2>/dev/null`)
- If the output file is empty or missing, report the error to the user
- If auth fails, tell the user to run `codex login`
- **If gpt-5.4 fails**, fall back to gpt-5.3-codex: replace `-m gpt-5.4` with `-m gpt-5.3-codex` and retry with the same flags
- **If the Bash tool times out**, the process was killed and output is lost. For Delegated tasks, re-run with `run_in_background: true`

## Context Engineering (Analysis Mode only)

When YOU are curating context for Codex, be a good context engineer:

| Context Type | Strategy |
|---|---|
| Architecture | Summarize in your own words, don't paste READMEs |
| Code | Only relevant snippets — hot paths, the specific functions under discussion |
| Dependencies | Names and versions, not full package.json |
| Question | Numbered, specific, with enough context to answer without filesystem access |
| Direction | Always include WHY we're doing this and what approach is locked in |

Don't artificially constrain output length unless the user wants brevity. If the question warrants a detailed answer, let Codex be detailed.

## Compaction Resilience

Output files are written to `~/.claude/subagent-results/` (persistent, not `/tmp/`). After context compaction:
- The output file still exists on disk — re-read it with the Read tool if needed
- CLAUDE.md compact instructions tell the compactor to preserve these file paths
- If you lose track of the file path, run: `ls -t ~/.claude/subagent-results/codex-output-* | head -1`

## Notes

- Codex is adversarial by nature — frame prompts with clear scope and direction to get constructive output
- Best for: code review, security analysis, bug hunting, verification, implementation, autonomous tasks
- Uses extended thinking (xhigh) for deepest reasoning
- GPT-5.4 is the latest model with improved reasoning and real-time steering
- `--ephemeral` prevents session file clutter — use for Quick and Analysis modes only, NOT Delegated
