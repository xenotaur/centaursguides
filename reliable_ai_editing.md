---
title: Reliable AI Editing
subtitle: Editor Contracts and Safe Workflows for AI-Assisted Code Updates
status: Living Document
intended_audience: Advanced developers, researchers, and tooling authors
last_updated: 2026-01-11
---

# Reliable AI Editing

## Purpose

> **TL;DR:** This document explains how to use AI assistants as *reliable code editors* by making authority, scope, and intent explicit.

Modern large language models are powerful collaborators for software development, but they are **not editors in the traditional sense**. In particular, conversational AI systems can produce *plausible but incorrect* code updates when authority boundaries are unclear. These failures are subtle, easy to miss in review, and especially dangerous in correctness‑critical codebases.

This document defines a set of **editor contracts, workflows, and prompt patterns** that allow developers to use AI assistants safely and predictably for code maintenance and evolution. It is intended as a **practical, reusable guide** rather than a theoretical discussion.

The goal is not to eliminate AI assistance, but to **restore trust** by making authority, scope, and intent explicit.

---

## The Core Problem

AI assistants operating in chat interfaces do not provide the same guarantees as traditional code editors or IDE refactoring tools. In particular:

- Code shown in earlier turns may no longer be treated as authoritative
- The model may attempt to *reconstruct* missing code from context
- Generated updates may be presented in a form that resembles real diffs or drop‑in replacements, even when based on inferred source

Because the output often *looks correct*, these errors can slip through review — especially in large or fast‑moving projects.

This document addresses that failure mode directly.

---

## A Mental Model Shift

This guide is built around a single conceptual change:

To use AI systems reliably for code updates, it is necessary to shift mental models:

> **From:** “The AI is my editor”  
> **To:** “The AI is a collaborator operating under an explicit editing contract”

An **editor contract** defines:

- When the AI is allowed to transform code
- What code is considered authoritative
- What kinds of changes are permitted
- How the AI must behave when those conditions are not met

The rest of this document describes how to implement these contracts in practice using today’s AI tooling.

---

## Authority Gating: No Code → No Edits

This section introduces the foundational invariant that all subsequent patterns rely on.

The single most important principle for reliable AI-assisted editing is **authority gating**:

> **An AI assistant may only generate diffs or drop-in replacements when the authoritative source code is explicitly present.**

In conversational interfaces, it is easy to assume that code shown in a previous turn, a prior canvas edit, or an earlier attachment is still “known” to the model. In practice, this assumption is unsafe. Models may:

- Lose access to earlier context
- Treat prior code as non-authoritative discussion material
- Attempt to reconstruct missing code from memory or description

When this happens, the AI may generate updates against *inferred* source while presenting them in a form that closely resembles a real edit. This is the failure mode authority gating is designed to prevent.

### The Rule

For any operation that produces:

- a drop-in replacement
- a unified diff
- or an updated version of existing code

**the exact authoritative code must appear verbatim in the same message** (inline, in the canvas, or as an attached file).

If the authoritative code is not present, the correct behavior is **refusal**, followed by a request for the code.

### Why Refusal Is the Correct Outcome

In traditional development workflows, tools do not edit files they cannot see. Authority gating brings the same discipline to AI-assisted coding. A refusal in this situation is not a failure; it is a safety feature.

By enforcing authority gating, you ensure that:

- All edits are grounded in real source
- Review diffs correspond to actual code
- Silent corruption via inferred edits is eliminated

### Practical Implication

As a user, this means adopting a simple habit:

> **Never ask for a drop-in replacement or diff unless you are prepared to provide the authoritative code in that turn.**

All subsequent patterns in this guide build on this principle.

---

## Editor Contracts: Explicit Guardrails for AI Editing

With authority gating established, the next step is to formalize it into reusable instructions.

An **editor contract** is a set of explicit instructions that define the conditions under which an AI assistant is permitted to modify code. Its purpose is to eliminate ambiguity about authority, scope, and expected behavior.

Unlike informal prompts or conversational context, an editor contract is **normative**: it specifies what the AI *must* and *must not* do, and it defines refusal as the correct outcome when preconditions are not met.

### What an Editor Contract Does

A well‑formed editor contract:

- Enforces **authority gating** (no edits without authoritative code)
- Prevents inferred or reconstructed source from being treated as real
- Distinguishes editing from suggestion or discussion
- Makes failure modes explicit and safe

In effect, the contract turns a conversational AI into a constrained editing tool.

### Where to Place Editor Contracts

Editor contracts can be applied at different scopes, depending on the project:

- **Project‑level instructions** (recommended): suitable when a repository or effort has consistent editing requirements
- **Chat‑level preamble**: useful for ad‑hoc or exploratory work
- **Per‑turn blocks**: appropriate for high‑risk or one‑off edits

Placing the contract at the project level provides persistent protection without repeated ceremony.

### Canonical Editor Contract Template

The following template can be copied verbatim into project instructions or a chat message:

```text
EDITOR CONTRACT — AUTHORITATIVE CODE TRANSFORMATION

You are acting as a code editor, not a code suggester.

Authority rules:
1. You may ONLY generate diffs or drop‑in replacements for code that appears verbatim in THIS MESSAGE (inline, canvas, or attached file).
2. You MUST NOT infer, reconstruct, guess, or approximate any part of the code.
3. If the authoritative code is not present in this message, you MUST STOP and ask me to paste or attach it.
4. If there is any ambiguity about what code is authoritative, you MUST ask before proceeding.

Output rules:
- If authority is satisfied, produce the requested edit.
- If authority is NOT satisfied, produce NO code and ask for the authoritative source.
- Do NOT present sample code as a drop‑in replacement.
- Do NOT present inferred code as real code.

A violation of these rules is an error.
```

This contract is intentionally strict. Its role is not to maximize convenience, but to maximize correctness and trust.

### Contracts vs. Suggestions

It is important to distinguish between:

- **Suggestion mode**: the AI proposes ideas, patterns, or example code
- **Edit mode**: the AI performs authoritative transformations on real source

Editor contracts apply **only** to edit mode. They do not restrict architectural discussion, debugging advice, or sample code generation, as long as those outputs are clearly labeled and not presented as real edits.

---

## Two‑Phase Workflow: Design → Apply

Editor contracts are most effective when paired with explicit turn structure.

Editor contracts and authority gating are most effective when paired with an explicit **two‑phase workflow**. This structure separates *thinking about changes* from *applying changes*, and makes authority transfer visible and deliberate.

The core idea is simple:

> **Phase 1:** Design and analysis (no editing authority)  
> **Phase 2:** Application and transformation (explicit editing authority)

### Phase 1: Design Mode (No Authority)

In Design Mode, the AI acts as a collaborator, reviewer, or sounding board. Typical activities include:

- Discussing architecture or tradeoffs
- Debugging by inspection
- Proposing multiple solution options
- Evaluating pros and cons

Code shown during this phase is **illustrative**, not authoritative. It may be partial, outdated, or intentionally simplified. Crucially, the AI must *not* produce drop‑in replacements or diffs in this phase.

A common Design Mode prompt looks like:

```text
Let’s stay in DESIGN MODE.
Discuss options and tradeoffs only.
Do not generate a drop‑in replacement yet.
```

Design Mode is intentionally flexible and conversational.

### Phase 2: Apply Mode (Authority Granted)

In Apply Mode, the user explicitly grants editing authority by:

1. Declaring the transition to apply/edit mode
2. Providing the **authoritative code verbatim**
3. Stating the desired editing constraints

Only in this phase may the AI generate a diff or drop‑in replacement.

A canonical Apply Mode prompt looks like:

````text
I choose option 2.

Please apply it using EDIT MODE.
Use the minimal‑diff constraints.

[EDITOR CONTRACT]

AUTHORITATIVE CODE:
```python
<exact code here>
```
````

If the authoritative code is missing or ambiguous, the correct behavior is refusal and a request for clarification.

### Why the Two‑Phase Workflow Works

This structure provides several benefits:

- Prevents accidental authority leakage across turns
- Makes it obvious when edits are *real* versus *illustrative*
- Reduces review burden by clarifying intent
- Mirrors established human workflows (design → implement)

Most importantly, it aligns conversational AI with the expectations developers already have of professional tooling.

### Authority Transfer Is Explicit

The boundary between phases is not implicit. It is marked by language, structure, and the presence of authoritative code. Treating this boundary explicitly eliminates the most dangerous failure mode: inferred edits presented as real ones.

---

## Editing Variants: Controlling the Semantics of Change

Once authority is granted, the remaining question is *how much* change is permitted.

Once authority has been granted via an editor contract and the workflow has entered **Apply Mode**, the remaining question is *what kind of change is desired*. Different situations call for different editing semantics, and making those semantics explicit greatly improves both safety and reviewability.

This section introduces common **editing variants**—standardized constraint sets that tell the AI *how aggressively* it may change the code.

### Variant 1: Minimal-Diff (Surgical Change)

**Use this variant when:**
- Fixing a bug
- Adding a small feature
- Making a narrowly scoped behavioral change
- Working in highly regulated or fragile code

**Intent:** Change as little as possible while achieving the functional goal.

A canonical minimal-diff constraint block:

```text
EDITING CONSTRAINTS — MINIMAL DIFF

- Make the smallest change necessary to implement the requested behavior.
- Preserve formatting, comments, ordering, and structure.
- Do not refactor unrelated code.
- The resulting diff should include ONLY lines that are semantically affected.
```

This variant minimizes review burden and reduces the risk of unintended side effects. It is the safest default for correctness-critical systems.

---

### Variant 2: Clean Refactor (Controlled Restructuring)

**Use this variant when:**
- Improving maintainability
- Paying down technical debt
- Modernizing style or patterns
- Refactoring internals while preserving external behavior

**Intent:** Allow internal restructuring while keeping external contracts stable.

A canonical clean-refactor constraint block:

```text
EDITING CONSTRAINTS — CLEAN REFACTOR

- You may refactor internal structure using best practices.
- Preserve the following as stable:
  - Public function signatures
  - Documented external behavior
- Internal variable names, helpers, and structure may change.
- Do not introduce new dependencies unless explicitly requested.
```

This variant trades minimal diffs for improved clarity and maintainability, while still protecting public APIs.

---

### Variant 3: Exploratory Rewrite (Low-Constraint)

**Use this variant when:**
- Prototyping
- Exploring alternative designs
- Working on throwaway or experimental code

**Intent:** Optimize for insight and clarity rather than diff minimality.

In this mode, the editor contract still applies, but editing constraints are deliberately relaxed. Outputs should be clearly labeled as exploratory and reviewed carefully before adoption.

---

### Choosing the Right Variant

Being explicit about editing variants has several benefits:

- Aligns the AI’s behavior with your review expectations
- Reduces accidental over-refactoring
- Makes diffs easier to reason about
- Documents intent for future readers

A useful rule of thumb:

> **Default to minimal-diff. Escalate to refactor only when you mean to.**

---

## One-Line Guards: Lightweight Safety for Fast Turns

This section covers lightweight safeguards for situations where full contracts feel too heavy.

While editor contracts and two-phase workflows provide the strongest guarantees, they can feel heavy for quick interactions. **One-line guards** are lightweight prompts that enforce the most critical safety property—authority gating—without requiring a full contract block.

One-line guards are best understood as **defensive rails**, not substitutes for full editor contracts. They are particularly useful when:

- You are iterating quickly
- The change is small but still correctness-critical
- You are following up on a prior design discussion
- You want to prevent accidental inferred edits

### Canonical One-Line Guard

This is the recommended default guard for most situations:

```text
Edit guard: If the authoritative code does not appear verbatim in this message, do NOT generate a drop-in replacement or diff; stop and ask me to paste or attach it.
```

This single sentence enforces the key invariant:

> **No authoritative code in this turn → no authoritative edit.**

### Ultra-Short Guard (Use with Care)

In very fast back-and-forths, a shorter guard may be sufficient:

```text
Edit guard: No authoritative code in this message → no drop-in replacement or diff.
```

This version is less explicit but still effective when paired with disciplined review.

### What One-Line Guards Do—and Do Not—Do

One-line guards:
- Prevent inferred or reconstructed source from being edited
- Force refusal when authority is missing
- Reduce the risk of silent, plausible corruption

They do **not**:
- Define editing semantics (minimal-diff vs. refactor)
- Replace full editor contracts for large or risky changes
- Provide the same clarity as explicit Apply Mode transitions

### Recommended Usage

A practical rule:

> **Use one-line guards for small, fast edits. Use full editor contracts for anything you would not approve without careful review.**

One-line guards work best as a habit—added reflexively before asking for a change—rather than as an afterthought.

---

## End-to-End Example Workflows

This section demonstrates how the patterns compose in realistic, complete interactions.

The patterns described so far become most useful when seen in complete, realistic workflows. This section walks through common end-to-end scenarios, illustrating how **design mode**, **authority gating**, **editor contracts**, and **editing variants** work together in practice.

These examples are intentionally schematic. The exact wording can be adapted to taste, but the structure and authority boundaries should be preserved.

### Example 1: Bug Fix with Minimal Diff

**Scenario:** A unit test is failing due to an edge-case bug. You want the smallest possible fix.

**Step 1 — Design Mode (Analysis Only):**

```text
Let’s stay in DESIGN MODE.
Here is the failing behavior and relevant code.
Please analyze the bug and propose options.
Do not generate a drop-in replacement yet.
```

The AI discusses possible causes and suggests a fix.

**Step 2 — Apply Mode (Authority Granted):**

````text
I choose option 1.

Please apply it using EDIT MODE.
Use the minimal-diff constraints.

[EDITOR CONTRACT]

AUTHORITATIVE CODE:
```python
<exact code here>
```
````

**Outcome:** A small, reviewable diff affecting only the relevant lines.

---

### Example 2: Feature Addition with Stable API

**Scenario:** You want to add a feature while preserving a public function signature.

**Step 1 — Design Mode:**

```text
Let’s stay in DESIGN MODE.
Please propose approaches for adding this feature while keeping the public API stable.
```

The AI proposes multiple designs.

**Step 2 — Apply Mode (Clean Refactor Variant):**

````text
I choose option 2.

Please apply it using EDIT MODE.
Use the clean-refactor constraints.

[EDITOR CONTRACT]

AUTHORITATIVE CODE:
```python
<exact code here>
```
````

**Outcome:** Internal restructuring with preserved external behavior.

---

### Example 3: Fast Follow-Up Edit with One-Line Guard

**Scenario:** After a prior change, you want to make a tiny adjustment.

```text
Edit guard: If the authoritative code does not appear verbatim in this message, do NOT generate a drop-in replacement or diff; stop and ask me to paste or attach it.

Please update the error message to be more descriptive.
```

If the code is present, the AI applies the change. If not, it correctly refuses and asks for the source.

---

## Failure Modes and Anti-Patterns

This section documents common mistakes that undermine reliability, even when users have read this guide.

Even with editor contracts and explicit workflows, certain patterns repeatedly lead to unsafe or misleading outcomes. This section catalogs **common failure modes and anti-patterns**, explains why they are dangerous, and describes the correct alternative.

These are drawn from real-world usage and tend to reappear precisely because they *feel* reasonable.

---

### Anti-Pattern 1: “Apply the Change We Just Discussed”

**Description:**
After a design discussion, the user says something like:

> “Okay, apply option 2.”

without re-providing the authoritative code.

**Why This Is Dangerous:**
- The AI may no longer have access to the exact code
- The model may infer or reconstruct the source
- The resulting output may look like a real diff or replacement

**Correct Pattern:**
Explicitly enter Apply Mode and paste or attach the authoritative code in the same turn.

---

### Anti-Pattern 2: Trusting Cross-Turn Code Memory

**Description:**
Assuming that code shown in a previous turn, canvas edit, or attachment is still authoritative.

**Why This Is Dangerous:**
- Conversational context is not a persistent file system
- Models may treat earlier code as illustrative only
- Silent loss of authority boundaries can occur

**Correct Pattern:**
Reassert authority every time by including the exact code in the edit request.

---

### Anti-Pattern 3: Accepting “Looks Right” Diffs

**Description:**
Reviewing an AI-generated diff that appears plausible and approving it without verifying it corresponds exactly to the provided source.

**Why This Is Dangerous:**
- Inferred source can closely resemble real code
- Small discrepancies can hide serious semantic changes

**Correct Pattern:**
Always confirm that the diff applies cleanly to the exact code you provided in that turn.

---

### Anti-Pattern 4: Mixing Design and Apply Modes

**Description:**
Asking for analysis, options, and a drop-in replacement in the same prompt.

**Why This Is Dangerous:**
- Blurs authority boundaries
- Encourages the model to overstep into inferred edits
- Makes intent unclear

**Correct Pattern:**
Separate design and application into distinct turns, with an explicit transition.

---

### Anti-Pattern 5: Overusing Refactor Variants

**Description:**
Requesting refactors by default, even for small or localized changes.

**Why This Is Dangerous:**
- Increases diff size and review burden
- Introduces unintended behavioral changes
- Masks the original intent of the change

**Correct Pattern:**
Default to minimal-diff variants and escalate to refactor only when explicitly intended.

---

### Anti-Pattern 6: Treating Refusal as Failure

**Description:**
Interpreting an AI refusal to edit (due to missing authority) as a limitation or error.

**Why This Is Dangerous:**
- Encourages loosening safety constraints
- Reinforces unsafe habits

**Correct Pattern:**
Treat refusal as confirmation that authority gating is working as intended.

---

### Warning Signs to Watch For

Stop and reassess if you see any of the following:

- The AI restates your code before editing it (especially from memory)
- The output includes code you do not recognize
- The diff touches areas unrelated to the stated intent
- The model apologizes and proceeds anyway without asking for the source

These are strong indicators of an authority or constraint mismatch.

---

## Quick Reference / Checklist

This checklist distills the guide into an operational pre-flight check.

This section provides a **concise, printable checklist** for day-to-day use. It is intended as a fast sanity check before asking an AI assistant to modify code.

### Before Asking for an Edit

- ☐ Am I asking for an **authoritative change** (diff or drop-in replacement), not just advice?
- ☐ Is the **exact authoritative code** available to paste or attach in this turn?
- ☐ Do I know whether I want a **minimal diff** or a **refactor**?

If any answer is “no,” stay in **Design Mode**.

---

### Granting Edit Authority (Apply Mode)

- ☐ Explicitly transition to **Apply / Edit Mode**
- ☐ Include the **editor contract** (project-level or per-turn)
- ☐ Paste or attach the **authoritative code verbatim**
- ☐ Specify the **editing variant** (minimal-diff, clean refactor, exploratory)

If the AI does not have the code in this turn, it should **refuse** and ask for it.

---

### Fast Edits (One-Line Guard)

For small, quick changes:

- ☐ Add a one-line guard before asking for the edit

```text
Edit guard: If the authoritative code does not appear verbatim in this message, do NOT generate a drop-in replacement or diff; stop and ask me to paste or attach it.
```

---

### Reviewing the Output

- ☐ Does the diff correspond to the **actual code I provided**?
- ☐ Are changes limited to the **intended semantic scope**?
- ☐ Are there any edits that look like refactors when I asked for minimal change?

If something looks surprising, assume an authority or constraint mismatch and stop.

---

### Golden Rules

> **No code in this turn → no authoritative edit**  
> **Refusal is a safety feature, not a failure**  
> **Default to minimal-diff; escalate only intentionally**

---

## Scope of This Guide

This section clarifies what this document is — and is not — intended to cover.

This guide focuses on:

- Drop‑in replacements and diffs
- Minimal semantic changes vs. intentional refactors
- Multi‑turn workflows involving design followed by application
- Preventing silent or inferred edits

It does **not** attempt to replace IDE‑level tooling, static analysis, or human review. Instead, it complements them by making AI behavior **explicit, reviewable, and predictable**.

---

In the sections that follow, we introduce concrete patterns — including reusable prompt blocks and one‑line guards — that can be pasted directly into project instructions or chat interactions to enforce reliable AI editing behavior.

