---
name: nestjs-patterns
description: NestJS structure and coding conventions — module boundaries, thin controllers, DTO validation, response mapping, errors, config, persistence and background work. Use when creating or changing a NestJS module, controller, service, DTO, entity, provider or worker, or when reviewing NestJS code.
---

# NestJS patterns

Apply these when writing NestJS code. If a rule conflicts with the project's CLAUDE.md, CLAUDE.md wins. Check the official docs (docs.nestjs.com, the ORM's docs for the pinned version) before using an API you are not sure of.

## Module boundaries

- One feature module per domain concept (`users`, `orders`, ...), each with its own controller, service, DTOs and tests next to the code.
- A module exports only the services other modules need. Never import another module's repository directly; go through its exported service.
- Shared, stateless helpers go in `common/`. Cross-cutting setup (pipes, filters, interceptors) is registered once in `main.ts` or `AppModule`.
- A circular import between modules is a design smell. Extract the shared part instead of reaching for `forwardRef`.

## Layers

| Layer | Owns | Must not |
|---|---|---|
| Controller | Routing, param parsing (`ParseIntPipe`, `ParseUUIDPipe`), status codes, calling one service method | Contain business rules, touch repositories, build queries |
| Service | Business rules, transactions, orchestration | Know about HTTP (no `Req`/`Res`) except throwing Nest HTTP exceptions |
| Pure functions | Calculations and decisions (pricing, eligibility, state transitions) | Do I/O. Keep them in plain `.ts` files so they unit-test without Nest |
| Entity | Persistence shape | Be returned from a controller |

## DTOs in, DTOs out

- Request DTOs use `class-validator` decorators. Register `new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true })` globally.
- Nested arrays need `@ValidateNested({ each: true })` plus `@Type(() => Child)`, or the children are not validated.
- Responses are mapped field by field into response DTOs or plain objects. Never return or spread an entity: hidden or lazy columns leak, and the API contract becomes tied to the schema.
- Data that must stay private (secrets, password hashes, internal flags) should be excluded at the query level (not selected), not only removed in the mapper.

## Errors

- Throw Nest exceptions with intent: `NotFoundException` (404), `ConflictException` (409, state conflicts such as paying an order twice), `BadRequestException` (400, semantically invalid input that passed shape validation).
- Messages are neutral and never reveal internal state or protected data.
- A global exception filter turns unknown errors into a 500 with a stable body and logs the stack. It never echoes raw DB errors to clients.

## Persistence

- Inject repositories with `@InjectRepository(Entity)`. Put a transaction in the service that owns the use case, and only where several writes must succeed or fail together.
- Guard state transitions with a conditional update (`WHERE id = ? AND status = ?`) and check the affected row count. Do not read, check and then write.
- Schema changes go through migrations with `synchronize: false`. Index foreign-key columns.
- Seed data is idempotent and runs from an npm script.

## Background and async work

- Work that must not block or fail the request (emails, webhooks) uses an outbox: write a PENDING row in the same transaction as the business change. A worker (`@nestjs/schedule`, or a queue in production) claims rows with a conditional update, then sends, retries with backoff, and marks a terminal FAILED state.
- Never fire-and-forget a promise from a request handler.
- The worker's tick method is public and callable directly so tests can drive it without timers.

## Config and operations

- `ConfigModule.forRoot` with a `validate` function or schema, so the app fails fast at boot on bad env. Commit `.env.example`, never `.env`.
- Use Nest's `Logger` with a context name, not `console.log`.
- Add `/health`, Swagger at `/docs` and `app.enableShutdownHooks()` when workers exist.

## Review checklist

- [ ] Controllers are thin; no repository in a controller.
- [ ] Every request body has a DTO; nested arrays are validated.
- [ ] No entity returned or spread into a response.
- [ ] Protected fields are never selected on public paths.
- [ ] State transitions are conditional updates with an affected-count check.
- [ ] Multi-write use cases are atomic; nothing async is fire-and-forget.
- [ ] Error messages are neutral; unknown errors become 500 without internals.
- [ ] New schema means a new migration plus FK indexes.
