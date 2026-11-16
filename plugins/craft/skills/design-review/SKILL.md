---
name: design-review
description: Turn the user's stated architecture decisions into a DESIGN.md (invariants, data model, API contract, flows, failure modes, trade-offs, out of scope) before code exists, and keep it in sync as decisions change. Use when starting a feature or service, or when the user asks to design, plan or document architecture. For an independent audit of code against the design, delegate to the craft reviewer agent instead.
---

# Design

The human owns the decisions. This skill structures and stress-tests them; it does not invent requirements. If a decision is missing, list it as an open question instead of choosing silently.

Write `DESIGN.md` from what the user said, in this order, no more than about one page:

1. **Goal.** One sentence.
2. **Invariants.** The few rules that must never break, numbered and testable ("X never appears in a response before Y"). Each names how it is enforced (query, constraint, transaction, test).
3. **Data model.** Entities with key fields, relations, unique constraints, indexes and which columns are sensitive.
4. **API contract.** Method, path, request, response, error codes. Mark which responses must exclude sensitive data.
5. **Flows.** The one or two non-trivial flows step by step (a mermaid sequence diagram if it helps), including where the transaction starts and ends.
6. **Failure modes.** For each external call or async step: what fails, what the user sees, what is retried, what is recorded.
7. **Trade-offs.** Decision → alternatives rejected → why, and what changes in production.
8. **Out of scope.** Said explicitly.

Then list open questions. Do not write code while designing.

When a decision changes later, update DESIGN.md in place with the final value; no history of earlier versions in the doc.

## Review

Do not self-review your own implementation. Hand the audit to the `reviewer` agent from this plugin (read-only, separate context), passing it the paths or diff to check, and report its verdict as-is.
