# DTTC — Design → Types → Tests → Code

DTTC is a strict development methodology that enforces a non-negotiable order of work.

**DTTC is written for AI coding agents.** It is a set of operating rules you give
to an agent (Claude Code, Cursor, Copilot, or any LLM that writes code) so that it
stops jumping straight to the implementation. A human applies this discipline when
rested; an agent applies it on every single task, at any hour. That is the point.

Humans can follow it too — the rules are the same. But the wording is deliberately
imperative, so it can be dropped as-is into an agent's instruction file.

## Flow

```
   Design  ──►  Types  ──►  Tests  ──►  Code
     (D)         (T)         (T)         (C)

   what must      the        the       the only
   be satisfied  shape     behavior   step that
                                      may change
```

Code is always the final step.

---

## Example

[**DTTC end to end — Python**](examples/python.md) — one small feature built
through all four steps, one test at a time, including a change that demonstrates
the adjustment hierarchy.

---

## Core Principle

> Code is a consequence.  
Intent, contracts, and behavior come before implementation.

---

## The Steps

### 1. Design (D)
Define intent and form **only where necessary**.

Design can be:
- partial (a section, a flow, a component)
- textual (description, rules, constraints)
- visual (HTML/JSX template, wireframe, image, sketch)
- skipped when not relevant (pure backend / logic)

**Choose based on context:**
- New UI → HTML/JSX template (full or scoped)
- Existing UI → localized adjustment
- Wireframe → sketch / ASCII / image
- Pure logic → text or skip

Design defines **what must be satisfied**, not how.

---

### 2. Types (T)
Define or adjust structure and contracts.

- Shapes, signatures, boundaries
- Types are hard constraints

---

### 3. Tests (T)
Define or adjust behavior.

- **1 test at a time**
- **1 implementation at a time**
- The active test defines the work

---

### 4. Code (C)
Implement only what is required to satisfy the active test.  
Nothing more.

---

## Working with Existing Code

The entry point may vary, **the order never changes**.

| Entry Point | Enforced Flow |
|------------|---------------|
| Code | Adjust implementation |
| Tests | Tests → Code |
| Types | Types → Tests → Code |
| Design | Design → Types → Tests → Code |

**Rules:**
- Enter as low as possible
- Always follow the strict order upward

---

## Adjustment Hierarchy

1. Adjust implementation (priority)
2. Avoid changing public contracts
3. Satisfy the top by modifying the bottom
4. Change contracts only when unavoidable

---

## Giving DTTC to an Agent

Put this file where your agent reads its instructions:

| Agent | File |
|-------|------|
| Claude Code | `CLAUDE.md` at the repo root |
| Cursor | `.cursor/rules/dttc.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Other | whatever system-prompt or rules file it supports |

Then state the contract explicitly:

> Follow DTTC. Announce the current step before each action.
> Never write implementation code before an active test exists.
> One test, one implementation, at a time.

**Why it works on an agent:** an LLM's default failure mode is writing plausible
code before the contract is settled. DTTC removes that option. The agent must
state the intent, fix the shape, express the behavior, and only then implement —
and each step is a checkpoint where you can stop it cheaply.

---

## When DTTC Does Not Apply

The discipline has a cost. Do not pay it when the work is exploratory:

- **Spikes** — you are writing code to learn what the types should be
- **Throwaway prototypes** — nothing survives, so no contract is worth fixing
- **Debugging** — the entry point is a failing behavior, not a design

In those cases, explore freely, then **throw the exploration away** and redo the
work under DTTC. The exploration was the design step.

---

## Summary

> DTTC is a strict discipline:  
never change the top while the bottom can still satisfy it.  
Code exists only to satisfy design, types, and tests.
