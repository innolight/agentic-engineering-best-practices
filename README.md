# Agentic Engineering Best Practices

This project discover best practices for Agentic Engineering from first principals.

Agentic engineering is the professional practice of orchestrating AI agents to implement software, shifting the human developer's role from manually writing code to directing specification, architecture, oversight, and quality control.

# Principals

## Treating Context as Finite, Degrading Resource

**The truth:** AI Agent performance is a function of what's in the context window — and performance degrades non-linearly as context fills. Managing context is not a secondary concern; it is the primary engineering challenge of agentic workflows.

**References**
- Anthropic's official Claude Code documentation states it directly: "Most best practices are based on one constraint: Claude's context window fills up fast, and performance degrades as it fills."
- The "lost-in-the-middle" phenomenon — where models lose factual precision near maximum capacity — is well-documented in research. 

### Contemporary practices


**Progressive Disclosure Principle.** Instead of a monolithic instruction file, layer instructions by scope — a root file (universal, high-signal rules), subdirectory files ( domain-specific rules, examples, references), and **on-demand skills/rules** (loaded only when relevant) - so agents receive only what they need, when they need it.

- For AGENT/CLAUDE.md: keep [< 200 lines](https://code.claude.com/docs/en/memory#write-effective-instructions); reference useful docs for progressive disclosure; craft universally applicable instructions such as behavioural guidelines, project identity (why, what, how) - [humanlayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md). 
- Keep SKILL.md < [500 lines](https://code.claude.com/docs/en/skills#add-supporting-files). Move detailed reference material to separate files (./examples, ./references, ./scripts)

**Proactively manage context.** Monitor context utilization and act, 
rather than waiting for degradation symptoms like hallucinations or ignored instructions.

- Scope each session to a single task; the "kitchen sink session" (mixing unrelated work) is the most common anti-pattern. A clean session with a precise prompt consistently outperforms a long session with accumulated corrections. 

- Delegate investigative work (reading many files, exploring options) to subagents.

- Practical heuristic: at 0–50% context window utilization work freely, at 50–70% pay attention, at 70–90% run compaction, at 90%+ a fresh session is mandatory.

- In Claude Code: `/clear` between unrelated tasks and start fresh, `/rename` to give current session a good name before clearing ,`/compact <focus>` with specific focus instructions, `/btw` for side-questions that don't enter history.


**The multi-tier memory model.** a *hot tier* (AGENT.md, compact, loaded every session), a *warm tier* (docs/ directory - enduring knowledge), and a *cold tier* (archived specs, plans, decision records).

**Reducing Input token**: 
 - [rtk](https://github.com/rtk-ai/rtk) - filter and compress CLI outputs
 - [headroom](https://github.com/chopratejas/headroom) - intercepts LLM requests, compresses the context, and forwards an optimized prompt

**Cutting Output tokens.** Instruction for communicating concisely. For example, [caveman](https://github.com/JuliusBrussee/caveman) skill.
