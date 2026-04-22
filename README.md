# Agentic Engineering Best Practices

This project aims to distill the best practices for Agentic Engineering from the First Principles. Agentic engineering is the professional practice of orchestrating AI agents to implement software, shifting the human developer's role from manually writing code to directing specification, architecture, oversight, and quality control.

# First Principles

1. [Context Engineering Principle](#context-engineering): treat context as finite cognitive workspace.
2. [Anti-hallucination Engineering Principle](#anti-hallucination-engineering): tame the probabilistic nature of LLMs through specification, decomposition, and verifications
3. Compound Engineering Principle: Each unit of engineering work should make subsequent units easier, not harder.
4. Secure By Design

## Context Engineering Principle

**Principle**: treat context as finite cognitive workspace.

**The truth:** AI Agent performance is a function of what's in the finite context window - and performance degrades non-linearly as context fills. Managing context is the primary engineering challenge of agentic workflows. 

**Research insights**
- **Reasoning rot**: instruction adherence degrades in three patterns - reasoning models plateau near-perfect adherence to ~150 instructions then collapse, frontier models decay linearly per instruction, small models fail catastrophically at 20-50 instructions. ([ref](references/researches/20250725-arxiv-2507.11538-how-many-instruction-can-llm-follow-at-once.md))
- **Primacy effect**: instructions earlier in context receive more attention. Instruction adherence dips lowest for instructions buried at the center. ([ref](references/researches/20250725-arxiv-2507.11538-how-many-instruction-can-llm-follow-at-once.md))
- **Silent omissions**: when overloaded, models silently drop instructions entirely rather than misapply them. ([ref](references/researches/20250725-arxiv-2507.11538-how-many-instruction-can-llm-follow-at-once.md))

See best practices for: [AGENTS.md](#agentsmd--claudemd) · [User Prompts](#user-prompts) · [Tools](#tools) · [Current Conversation](#current-conversation) · [Knowledge](#knowledge) · [Memories](#memories)

<a id="agentsmd--claudemd"></a>**📜 AGENTS.md / CLAUDE.md**

| Best Practices | Ref |
|---|---|
| **Less (instructions) is more** (adherence). Have < 200 lines | [claude](https://code.claude.com/docs/en/memory#write-effective-instructions) |
| **Craft concise, universally applicable instructions**: behavioural guidelines, project identity WHY (purpose), WHAT (tech stack, project structure), HOW (commands/instructions to do meaningful work). | [humanlayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md) |
| **Leverage progressive disclosure** by telling agents how to find important information. | |
| **Skip what the model already knows** or what linters enforce. | [humanlayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md) |

<a id="user-prompts"></a>**✏️ User Prompts**

| Best Practices | Ref |
|---|---|
| **Clear prompting.**. Provide as much context as possible. Specify Goal (what to build), Context (files, folders, docs, examples...), Constrains (standards, architecture, safety requirements,...), and Success Criterias. Ambiguity means agent will fill gaps with hallucinations. | [Codex](https://developers.openai.com/codex/learn/best-practices#strong-first-use-context-and-prompts) |
| **Interview-me pattern**. Have the agent ask detailed questions about requirements, edge cases, tradeoffs. Invert the typical prompt-response dynamic. | [Claude](https://code.claude.com/docs/en/best-practices#let-claude-interview-you) |

<a id="tools"></a>**🛠 Tools** - the executable capabilities agents invoke: bash, MCP servers, CLIs.

| Best Practices | Ref |
|---|---|
| Replace instructions with deterministic hooks when possible. Hooks for linting, formatting, type-checking execute regardless of what the agent "remembers". |  |
| Optimise tool output to be token efficient. |  |
| Filter and compress verbose CLI outputs before they enter context. | [rtk](https://github.com/rtk-ai/rtk) |


<a id="current-conversation"></a>**💬 Current Consersation** - the fastest-growing part of context

| Best Practices | Ref |
|---|---|
| **Scope one task per session**. Avoid the "kitchen sink session" anti-pattern. A clean session with a precise prompt consistently outperforms a long session with accumulated corrections. | |
| **Proactively manage context**. Heuristic: at 0–50% usage work freely, 50–70% pay attention, 70–90% compact/hands-off, 90%+ start fresh. Don't wait for degradation symptoms like hallucinations or ignored instructions. | |
| **Delegate to subagents** .e.g. investigative/research work. Their tool outputs and reasoning stay in separate context. Your main session context gets the result back. | [anthropic-doc](https://code.claude.com/docs/en/best-practices#use-subagents-for-investigation) |
| **Rewind conversation**. After a failed attempt, rewind to jump back to any previous message and re-prompt from there with what you learned. | [anthropic-team](./references/anthropic-team/20260416-claude-thariq-tips.md) · [anthropic-doc](https://code.claude.com/docs/en/best-practices#rewind-with-checkpoints) |
| [Experimental] Intercept and compress the accumulated session context before it reaches the model. | [headroom](https://github.com/chopratejas/headroom) |
| [Experimental] Instruct agents to communicate concisely. | [caveman](https://github.com/JuliusBrussee/caveman) |

<a id="knowledge"></a>**📚 Knowledge**

Domain-specific, project-specific, and technical information agents need: internal/external documentation, code examples, API specs, architectural decisions.

| Best Practices | Ref |
|---|---|
| Maintain knowledge as living artifacts. Stale knowledge harms more than missing knowledge. Update docs when agent failures reveal incorrect or outdated information. | |
| Use Architectural Decision Records. | |

<a id="memories"></a>**💾 Memories**

Persistent state that accumulates across sessions: learned preferences, project context, past decisions.

| Best Practices | Ref |
|---|---|
| Leverage a multi-tier memory system: hot tier (loaded every session, <200 lines), warm tier (on-demand docs/), cold tier (archived records). Match access frequency to storage tier to prevent context bloat. | |
| Promote frequently-needed insights to hot tier, enduring references to warm tier, archive the rest. Prevent knowledge accumulation from becoming context pollution. | |
| Enforce structure on what gets stored (facts, decisions, patterns, warnings). Unstructured dumps of raw outputs cause more retrieval problems than they solve. | |
| Scratchpad extraction at task boundaries: when transitioning tasks or sessions, extract critical facts into condensed summaries. This extends agent operational lifespan before attention degradation. | |

Coding-agent practicalities:
- In Claude Code: `/clear` between unrelated tasks and start fresh, `/rename` to give current session a good name before clearing, `/compact <focus>` with specific focus instructions, `/btw` for side-questions that don't enter history.

## Anti-hallucination Engineering

_Principle_: tame the probabilistic/stochastic nature of LLMs through specification, decomposition, and verifications.

_The truth_: LLMs optimize for plausibility, not correctness. When a prompt is ambiguous, the model fills gaps with common patterns from training data rather than project-specific intent. The quality of agent output is bounded by the precision of the input specification and the rigor of the verification loop. Premature coding is the dominant failure mode.

_Research insights_:
- The Columbia University DAPLab found that agents "prioritize runnable code over correctness." They generate something that compiles but silently ignores existing patterns, duplicates logic, or violates conventions.
- The ACONIC framework (Wei et al., 2025) demonstrated that formalizing task specifications improved agent performance by **10–40 percentage points** on complex tasks.
- Teams using AI without quality guardrails report a **35–40% increase in bug density** within six months (Qodo 2025).
- Agents experience performance degradation after 35 minutes of human-equivalent time, with non-linear failure curve: doubling task duration quadruples the failure rate (Zylos Research, 2026).
- 76% of developers are in the "red zone" (frequent hallucinations, low confidence) yet only 48% always check AI code before committing (Qodo 2025).
- The "rubber-stamp" failure mode: an agent reviewing code systematically approves flawed logic because agreement is the path of least mathematical resistance in its training distribution.

### Pillars of Best Bractices 

This principle decomposes into three pillars of practice: Spec-Driven Development for precise specification at the input, structured decomposition in the middle, and rigorous verification at the output.

```
Spec-Driven Development → defines what to verify and what to decompose
         ↓                              ↓
Structured Decomposition → creates subtasks small enough to verify
         ↓
Verification-Driven Autonomy → confirms each subtask matches the spec
```

### Spec-Driven Development (SDD)

The agent should never generate code until what-to-build is unambiguous.
ThoughtWorks called SDD "one of the most important practices to emerge in 2025." 

#### Best practices

| Best Practices | Ref |
|---|---|
| **Plan-then-implement separation.** The industry has converged on the canonical workflow: **Explore → Plan → Implement → Verify**. During exploration, the agent reads code and builds understanding without making changes. During planning, it produces a written artifact that the engineer reviews and annotates. Only after plan approval does implementation begin. | |
| **Interview-me pattern**. Have the agent ask detailed questions about requirements, edge cases, tradeoffs. Invert the typical prompt-response dynamic. | [Claude](https://code.claude.com/docs/en/best-practices#let-claude-interview-you) |
| **Reference-pattern anchoring.** Give agents concrete examples ("follow the pattern in X") rather than abstract descriptions. Concrete examples provide stronger conditioning signals than abstract instructions. | |
| **Specification as a living artifact.** Martin Fowler's team identifies three maturity levels: spec-first (written before coding), spec-anchored (maintained during development), and spec-as-source (the specification *is* the primary artifact, code is derived) | [martinfowler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) |

### Structured Decomposition

**The idea**: Agent failure rates scale non-linearly with task complexity. Breaking complex work into bounded, independently verifiable subtasks is the primary mechanism for keeping failure rates manageable. Beware, decomposition has diminishing returns and should match task complexity.

#### Best practices

| Best Practices | Ref |
|---|---|
| **Right-sized decomposition.** If you can describe the desired diff in one sentence, skip the planning overhead. If the task touches multiple files or you're uncertain about the approach, decompose and plan. Taxonomy: narrow tasks (agent handles alone), context-dependent tasks (needs project context), architectural tasks (requires human guidance). | |
| **Verifiable subtask boundaries.** Each subtask should have clear success criteria that can be checked independently. This transforms one high-stakes verification at the end into many low-stakes verifications throughout. | |
| **Planner-worker separation.** The planner generates a multi-step plan with dependencies; workers execute individual steps in bounded context. Achieves up to 90% cost reduction by using frontier models only for planning. | |
| **File-based state persistence.** Use plan.md/progress.md to bridge across context windows. The agent reads the spec, checks the plan for the current task, executes, updates the plan, and terminates. | |

### Verification-Driven Autonomy

**The idea**: LLMs cannot reliably self-evaluate their own outputs. Environmental feedback - compilers, tests, linters - provides ground truth that no amount of internal reasoning can substitute. Autonomy must be earned through deterministic verification, not assumed through stochastic self-reflection.

#### Best practices

| Best Practices | Ref |
|---|---|
| **Multi-layer verification pipeline.** No single verification method catches all failure modes. Multiple layers of deterministics verification are needed: type checking → (unit/integration/e2e) tests → static analysis → formatting → linting → security scanning → agents review → human review. | |
| **TDD - Test Driven Development.** Tests anchor the specification, preventing agents from drifting into plausible-but-wrong code. Tests creates automated feedback loop for agent to self-correct against each iteration. Tests enforce creation of testable, maintainable design. | |
| **Separate agent reviewers** (same or different models) catch problems more reliably than self-assessment. A fresh context window avoids biases accumulated during implementation. | |
| **Continuous verification during implementation.** Run type checking, linting, targeted tests,... after each meaningful change, not just at the end. Catch drift early before errors compound. Hooks provide deterministic, automated enforcement that doesn't depend on what the agent "remembers". | |
| **Atomic commits**: each commit represents a single, self-contained, verified logical change to the codebase. | |

