# Back-end Guidelines

## Stack

- Fastify 5
- REST API
- Prisma 5 (PostgreSQL) + Zod validation
- Deployed on AWS ECS Fargate

## Directory Layout

|- apps/
| |- backend/
| | |- prisma/ # Prisma schema & migrations (infra-only)
| | |- src/
| | | |- core/ # Business logic (pure domain layer)
| | | | |- entities/ # Domain models
| | | | |- use-cases/ # Application-specific business rules
| | | | |- ports/ # Interfaces (inbound/outbound)
| | | |- infrastructure/ # Adapters (implements ports)
| | | | |- db/ # Prisma or other DB adapter
| | | | |- cache/ # Redis adapter
| | | | |- storage/ # S3 or other file storage adapter
| | | |- framework/ # Framework glue (e.g. Fastify setup)
| | | | |- server/ # Entry point setup (e.g. fastify instance, hooks)
| | | | |- plugins/ # Fastify plugins, middleware
| | | |- routes/ # I/O adapters
| | | | |- v1/ # REST handlers/controllers
| | | |- clients/ # Outgoing SDKs or HTTP wrappers (external services)
| | | |- resources/ # Bootstrapped singletons (db, redis instances, env config)
| | | |- types/ # Shared DTOs, schema types, enums
| | | |- utils/ # Pure helpers, no side effects
| | |- tests/
| | | |- unit/ # Isolated tests (use-cases, entities)
| | | |- integration/ # End-to-end or infra-aware tests

## API & Error-handling

- Use **RESTful** verbs & status codes.
- Validate every request body/query/param with Zod.
- Central error handler → JSON `{ error, message }`.
- Generate shared types between frontend and backend.

## Security Checklist

- Auth-guard protected routes.
- Never leak stack traces.
- Sanitize external input (SQL/NoSQL/XSS).

## Testing

- Unit: pure core & services (mock ports) → `tests/unit/`
- Integration: start Fastify + test DB → `tests/integration/`
- Run via `pnpm test`.
