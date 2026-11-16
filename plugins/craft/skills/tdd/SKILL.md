---
name: tdd
description: Test-driven development loop for TypeScript/NestJS — write the failing test first, prove it fails for the right reason, make it pass with the smallest change, refactor on green. Use when implementing business logic, calculation rules, state transitions or async workflows, when the user says "TDD", "test first" or "red-green", or when adding tests to a NestJS service.
---

# TDD

## The loop (one behaviour at a time)

1. **List behaviours.** Before any code, write the behaviours as a short list of test names in plain language, including edge cases and boundaries. Show the list and wait for confirmation if the spec is ambiguous.
2. **Red.** Write one test. Run it. It must fail, and fail for the expected reason (an assertion, not a typo, a missing import or a compile error elsewhere). Paste the failure line.
3. **Green.** Write the smallest production change that passes. Run the suite again.
4. **Refactor.** Clean names and duplication with tests green. Run again.
5. Repeat for the next behaviour. Report a table: behaviour → test → status.

Hard rules:
- Never edit or delete a test to make it pass. If the test is wrong, say why and ask.
- Never write production code without a red test that needs it.
- No `.skip`, `.only` or snapshot tests for logic.

## What to test where

| Kind | Tool | Use for |
|---|---|---|
| Pure unit | Plain Jest, no Nest | Calculations, mappers, decision rules. Start here. Push logic into pure functions so most tests live here. |
| Service unit | `Test.createTestingModule` with mocked providers | Orchestration: correct calls, error paths, state guards. |
| Integration / e2e | Real app + real DB (e.g. SQLite file or in-memory) + `supertest` | Queries mocks cannot verify: aggregates, raw SQL, dates, transactions, migrations, HTTP contract and leak checks. |

## Choosing cases

- **Boundaries:** test both sides of every threshold (for 50 and 70 cut-offs: 49, 50, 69, 70), plus 0, empty, max.
- **Partitions:** one case per equivalence class, not ten cases from the same class.
- **Errors:** every thrown exception has a test asserting its type (404/409/400), not only that "it throws".
- **Invariants:** a test that the protected thing never happens (a field never appears in a response, an order cannot be paid twice).
- **Async/fault tolerance:** the business operation succeeds when the side effect fails; a retry happens; a terminal failure is recorded; a claimed item is not processed twice.

## NestJS test patterns

```ts
const module = await Test.createTestingModule({
  providers: [
    OrdersService,
    { provide: getRepositoryToken(Order), useValue: { findOne: jest.fn(), update: jest.fn() } },
    { provide: DataSource, useValue: { transaction: jest.fn((cb) => cb(entityManagerMock)) } },
  ],
}).compile();
```

- Mock at the boundary you own (repository, external client), not deep internals.
- Workers and schedulers: call the tick method directly. Use fake timers only if time itself is the behaviour; never `sleep` in tests.
- Arrange-Act-Assert, one behaviour per test, test names describe behaviour (`returns 409 when the order is already paid`).
- Build test data with small factory helpers, not giant shared fixtures.

## Done means

All tests green, type-check clean, and the behaviour table shows each listed behaviour covered. Report the command output, not a summary of it.
