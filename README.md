# DTTC — Design → Types → Tests → Code

DTTC is a strict development methodology that enforces a non-negotiable order of work.

## Flow


Code is always the final step.

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

## Summary

> DTTC is a strict discipline:  
never change the top while the bottom can still satisfy it.  
Code exists only to satisfy design, types, and tests.
