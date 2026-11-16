---
name: design-review
description: Architecture and design for backend services, in two modes. "design" turns the user's stated decisions into a DESIGN.md (invariants, data model, API contract, failure modes, trade-offs) before code exists. "review" is a read-only audit of code or a plan against that design and a rubric, reporting only real findings with file:line. Use when starting a feature or service, when the user asks to design, plan or document architecture, or asks to review design, architecture or code quality.
---

# Design review

The human owns the decisions. This skill structures and stress-tests them; it does not invent requirements. If a decision is missing, list it as an open question instead of choosing silently.

## Mode: design

Write `DESIGN.md` from what the user said, in this order, no more than about one page:

1. **Goal.** One sentence.
2. **Invariants.** The few rules that must never break, numbered and testable ("X never appears in a response before Y"). Each names how it is enforced (query, constraint, transaction, test).
3. **Data model.** Entities with key fields, relations, unique constraints, indexes and which columns are sensitive.
4. **API contract.** Method, path, request, response, error codes. Mark which responses must exclude sensitive data.
5. **Flows.** The one or two non-trivial flows step by step (a mermaid sequence diagram if it helps), including where the transaction starts and ends.
6. **Failure modes.** For each external call or async step: what fails, what the user sees, what is retried, what is recorded.
7. **Trade-offs.** Decision → alternatives rejected → why, and what changes in production.
8. **Out of scope.** Said explicitly.

Then list open questions. Do not write code in this mode.

## Mode: review (read-only)

Do not edit files. Read the design, then the code it covers.

Rubric:
- **Invariants:** each one is enforced in code and covered by a test. Trace the path end to end; do not trust names.
- **Boundaries:** dependencies point inward (controller → service → pure logic / repository). No HTTP in services, no queries in controllers, no cross-module repository access.
- **Atomicity and concurrency:** multi-write use cases are atomic; state transitions are guarded against double execution; no fire-and-forget.
- **Data exposure:** sensitive fields are excluded at query level and absent from DTOs; errors do not leak internals.
- **Failure handling:** every side effect has a defined failure path and recorded status.
- **Simplicity:** no abstraction without a second use; no speculative config or layers. Flag over-engineering as firmly as gaps.
- **Drift:** code that contradicts DESIGN.md, or design claims no code implements.

Output:
- Only **Critical** (breaks an invariant, loses data, security) and **Important** (likely bug, wrong boundary, missing test for an invariant) findings. No style nits.
- Each finding: severity, `file:line`, the concrete failure scenario (input → wrong result), and the smallest fix.
- End with a verdict line: `PASS` or `FAIL (n critical, m important)`.
- If nothing survives, say PASS and list what you checked.
