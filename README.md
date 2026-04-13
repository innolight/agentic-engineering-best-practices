# Agentic Engineering Best Practices

This project aims to distill the best practices for Agentic Engineering from the First Principles. Agentic engineering is the professional practice of orchestrating AI agents to implement software, shifting the human developer's role from manually writing code to directing specification, architecture, oversight, and quality control.

# First Principles

1. [Context Engineering](#context-engineering) - treating context as finite cognitive workspace
2. [Anti-hallucination Engineering](#anti-hallucination-engineering) - taming the probabilistic nature of LLMs
3. Compound Engineering - enabling knowledge to compound in actions
4. Secure By Design


## Context Engineering

**The truth:** AI Agent performance is a function of what's in the finite context window - and performance degrades non-linearly as context fills. Managing context is not a secondary concern; it is the primary engineering challenge of agentic workflows. 

**Research insights**
- Anthropic's official docs: "Most best practices are based on one constraint: Claude's context window fills up fast, and performance degrades as it fills."
- The "lost-in-the-middle" phenomenon — where models lose factual precision near maximum capacity — is well-documented in research. 

### Best practices

**Progressive disclosure**: instead of a monolithic instruction file, layer instructions by scope — a root file (universal, high-signal rules), subdirectory files ( domain-specific rules, examples, references), and **on-demand skills/rules** (loaded only when relevant) - so agents receive only what they need, when they need it.

- For AGENT/CLAUDE.md: keep [< 200 lines](https://code.claude.com/docs/en/memory#write-effective-instructions); reference useful docs for progressive disclosure; craft universally applicable instructions such as behavioural guidelines, project identity (why, what, how) - [humanlayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md). 
- Keep SKILL.md < [500 lines](https://code.claude.com/docs/en/skills#add-supporting-files). Move detailed reference material to separate files (./examples, ./references, ./scripts)

**Proactively manage context**: Monitor context utilization and act, rather than waiting for degradation symptoms like hallucinations or ignored instructions.

- Scope each session to a single task; the "kitchen sink session" (mixing unrelated work) is the most common anti-pattern. A clean session with a precise prompt consistently outperforms a long session with accumulated corrections. 

- Delegate investigative work (reading many files, exploring options) to subagents.

- Practical heuristic: at 0–50% context window utilization work freely, at 50–70% pay attention, at 70–90% run compaction, at 90%+ a fresh session is mandatory.

- In Claude Code: `/clear` between unrelated tasks and start fresh, `/rename` to give current session a good name before clearing ,`/compact <focus>` with specific focus instructions, `/btw` for side-questions that don't enter history.


**The multi-tier memory model.** a *hot tier* (AGENT.md, compact, loaded every session), a *warm tier* (docs/ directory - enduring knowledge), and a *cold tier* (archived specs, plans, decision records).

**Reducing Input token**: through proxies:
 - [rtk](https://github.com/rtk-ai/rtk) - filter and compress CLI outputs
 - [headroom](https://github.com/chopratejas/headroom) - intercepts LLM requests, compresses the context, and forwards an optimized prompt

**Reducing Output tokens** by instruction for communicating concisely. [caveman](https://github.com/JuliusBrussee/caveman) skill is an fun experiment at this.

**Reference-pattern anchoring.** Agents perform dramatically better when given concrete examples from the existing codebase rather than abstract descriptions.

## Anti-hallucination Engineering

**The truth:** LLMs optimize for plausibility, not correctness. When a prompt is ambiguous, the model fills gaps with common patterns from training data rather than project-specific intent. Every practitioner report identifies premature coding as the dominant failure mode. The quality of agent output is bounded by the precision of the input specification and the rigor of the verification loop.

**Research insights**
- The Columbia University DAPLab found that agents "prioritize runnable code over correctness" — they generate something that compiles but silently ignores existing patterns, duplicates logic, or violates conventions.
- The ACONIC framework (Wei et al., 2025) demonstrated that formalizing task specifications improved agent performance by **10–40 percentage points** on complex tasks.
- Teams using AI without quality guardrails report a **35–40% increase in bug density** within six months (Qodo 2025).
- Agents experience performance degradation after 35 minutes of human-equivalent time, with non-linear failure curve: doubling task duration quadruples the failure rate (Zylos Research, 2026).

This principle decomposes into three pillars of practice: Spec-Driven Development for precise specification at the input, structured decomposition in the middle, and rigorous verification at the output.

```
Spec-Driven Development → defines what to verify and what to decompose
         ↓                              ↓
Structured Decomposition → creates subtasks small enough to verify
         ↓
Verification-Driven Autonomy → confirms each subtask matches the spec
```


### Spec-Driven Development (SDD)

The agent should never generate code until what-to-build is unambiguous. Specifications are first-class artifacts that drive both implementation and verification — shifting the human role from writing code to writing precise intent. ThoughtWorks called SDD "one of the most important practices to emerge in 2025." 

**Key insight:** "The issue isn't the agent's coding ability, but our approach. We treat coding agents like search engines when we should be treating them like literal-minded pair programmers." — GitHub Spec Kit

#### Best practices

**Plan-then-implement separation.** The industry has converged on the canonical workflow: **Explore → Plan → Implement → Verify**. During exploration, the agent reads code and builds understanding without making changes. During planning, it produces a written artifact that the engineer reviews and annotates. Only after plan approval does implementation begin. The critical discipline is preventing the agent from writing code prematurely.

**The interview pattern.** The agent asks the developer detailed questions about requirements, edge cases, and tradeoffs before producing a specification document. This inverts the typical prompt-response dynamic and produces far richer specifications.

**Reference-pattern anchoring.** Agents perform dramatically better when given concrete examples from the existing codebase ("follow the pattern in X") rather than abstract descriptions. Concrete examples provide stronger conditioning signals than abstract instructions.

**Specification as a living artifact.** Martin Fowler's team identifies three maturity levels: spec-first (written before coding), spec-anchored (maintained during development), and spec-as-source (the specification *is* the primary artifact, code is derived) - [article](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)

### Structured Decomposition

Agent failure rates scale non-linearly with task complexity. Breaking complex work into bounded, independently verifiable subtasks is the primary mechanism for keeping failure rates manageable — but decomposition has diminishing returns and should match task complexity.

**Key insight:** SWE-bench Pro (multi-file, real-world complexity) caps the best models at ~23% vs ~79% on SWE-bench Verified. The gap is complexity. ChatDev's "chat chain" decomposition raised code quality from 0.15 to 0.40. But excessive decomposition increases coordination overhead (Amazon Science) and unstructured multi-agent networks amplify errors by up to 17.2x (Google, 2025).

#### Best practices

- **Right-sized decomposition.** If you can describe the desired diff in one sentence, skip the planning overhead. If the task touches multiple files or you're uncertain about the approach, decompose and plan. Taxonomy: narrow tasks (agent handles alone), context-dependent tasks (needs project context), architectural tasks (requires human guidance).

- **Verifiable subtask boundaries.** Each subtask should have clear success criteria that can be checked independently. This transforms one high-stakes verification at the end into many low-stakes verifications throughout.

- **Planner-worker separation.** The planner generates a multi-step plan with dependencies; workers execute individual steps in bounded context. Achieves up to 90% cost reduction by using frontier models only for planning.

- **File-based state persistence.** Use plan.md/progress.md to bridge across context windows. The agent reads the spec, checks the plan for the current task, executes, updates the plan, and terminates.

### Verification-Driven Autonomy

**The idea**: LLMs cannot reliably self-evaluate their own outputs. Environmental feedback - compilers, tests, linters - provides ground truth that no amount of internal reasoning can substitute. Autonomy must be earned through deterministic verification, not assumed through stochastic self-reflection.

**Research insight**
- 76% of developers are in the "red zone" — frequent hallucinations with low confidence — yet only 48% always check AI code before committing (Qodo 2025).
- The "rubber-stamp" failure mode: an agent reviewing code systematically approves flawed logic because agreement is the path of least mathematical resistance in its training distribution.

#### Best practices

- **Multi-layer verification pipeline.** No single verification method catches all failure modes. Engineer multiple layers of deterministics verification: type checking → (unit/integration/e2e) tests → static analysis → formatting → linting → security scanning → human review. 

- **TDD - Test Driven Development.** Writing tests that assert the specification before implementation. Writing codes before write tests, you risk getting tests that assert the buggy behavior.

- **Separate agent reviewers** (same or different models) catch problems more reliably than self-assessment. A fresh context window avoids biases accumulated during implementation.

- **Continuous verification during implementation.** Run type checking, linting, formatting, and targeted tests after each meaningful change, not just at the end. Catch drift early before errors compound.
    - Hooks provide deterministic, automated enforcement that doesn't depend on what the agent "remembers".

- **Atomic commits.** Commit after each verified, self-contained change. This creates rollback points, makes each change independently reviewable, and prevents the "big bang" merge where errors are entangled across files.

