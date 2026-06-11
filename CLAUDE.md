# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
at specs/014-coupon-assign-owner/plan.md
<!-- SPECKIT END -->

## Repository Structure

This is a monorepo with three independent services:
- `Latinhouse.Be/`: Backend (Spring Boot 4.0 / Java 25, Gradle)
- `Latinhouse.Fe/`: Frontend (currently empty scaffold, Next.js 16 / React 19 planned)
- `Latinhouse.Admin/`: Admin service (Next.js 16 / React 19) — admin dashboard for profiles, lessons, orders, coupons

Feature work is tracked under `specs/<NNN-feature-name>/` (Spec Kit workflow: spec.md, plan.md, tasks.md).

## Agent Delegation Rules

- Latinhouse.Be 관련 작업 → backend-agent
- Latinhouse.Fe 관련 작업 → front-agent
- Latinhouse.Admin 관련 작업 → admin-agent

Each subagent's working directory is the corresponding service folder; commands should be run with `cd` into that folder first.

## Backend (Latinhouse.Be)

### Commands
Run from `Latinhouse.Be/`:
```bash
./gradlew build              # build (compiles + runs tests)
./gradlew test                # run all tests
./gradlew test --tests "com.latinhouse.api.lesson.application.service.GetLessonServiceTest"  # single test class
./gradlew bootRun              # run the app (H2, in-memory, ddl-auto: create-drop)
```

### Architecture: Hexagonal (Ports & Adapters)

Each domain (`coupon`, `lesson`, `order`, `profile`) under `com.latinhouse.api.<domain>` follows:
```
<domain>/
├── adapter/
│   ├── in/web           # Controller, {Domain}WebRequest/WebResponse, {Domain}WebMapper
│   └── out/persistence  # JPA Entity, Repository impl, Persistence Adapter, {Domain}PersistenceMapper
├── application/
│   ├── port/in           # UseCase interface, {Domain}AppRequest/AppResponse, {Domain}AppMapper
│   ├── port/out          # Port interfaces (e.g. SaveProfilePort)
│   └── service           # UseCase implementations
└── domain                # pure domain objects, enums (no external deps)
```

Dependency direction is fixed: `Adapter → Application → Domain`. Domain layer must have zero JPA/Spring/HTTP dependencies. Application layer must never touch JPA entities or raw HTTP types directly.

### Two-DTO Pattern (Request/Response separation)

| Class | Location | Role |
|---|---|---|
| `{Domain}WebRequest` | `adapter/in/web/` | HTTP input. Bean Validation (`@NotBlank`, `@Pattern`, etc). Primitive types (`String`). |
| `{Domain}AppRequest` | `application/port/in/` | App-layer command. Domain types (`Sex`, `LocalDate`, etc). No validation annotations. |
| `{Domain}AppResponse` | `application/port/in/` | App-layer result, returned by UseCase. |
| `{Domain}WebResponse` | `adapter/in/web/` | HTTP response, returned by Controller as `ResponseEntity<{Domain}WebResponse>`. |

Conversion logic lives in dedicated, non-instantiable (private constructor), static-method-only Mapper classes — not in DTOs:
- `{Domain}WebMapper` (adapter/in/web): `WebRequest → AppRequest`, `AppResponse → WebResponse`. Primitive→domain type conversion (e.g. `Sex.valueOf()`) happens here.
- `{Domain}AppMapper` (application/port/in): `AppRequest → Domain object`, `Domain object → AppResponse`. Business defaults (e.g. `isInstructor(false)`) applied here.
- `{Domain}PersistenceMapper` (adapter/out/persistence): `Domain object ↔ JPA Entity`.

### API & Error Conventions

- API contract is defined in `docs/api-spec.md` — implementation must match exactly. If they diverge, fix the spec first, then the code.
- Domain model is documented in `docs/data-model.md`.
- All error responses use this unified shape:
  ```json
  { "status": 400, "errors": [{ "field": "필드명", "message": "에러 메시지" }] }
  ```
- All endpoints must be documented with `@Tag` and `@Operation` (springdoc/Swagger).
- Validation annotations only on `{Domain}WebRequest`, never on `{Domain}AppRequest`.

### Spring Boot 4.0 Test Gotchas

Spring Boot 4.0 reorganized test infrastructure vs 3.x:
- `@WebMvcTest` moved to `org.springframework.boot.webmvc.test.autoconfigure.WebMvcTest` (requires `testImplementation 'org.springframework.boot:spring-boot-webmvc-test'`).
- `ObjectMapper` is not auto-configured in `@WebMvcTest` — declare `new ObjectMapper()` as a static field instead of `@Autowired`.
- `@MockitoBean` (from `org.springframework.test.context.bean.override.mockito.MockitoBean`) replaces `@MockBean`.

### Quality Gates (must pass before commit)

1. No compile errors
2. All tests pass
3. Implementation matches `docs/api-spec.md`
4. All endpoints documented with `@Tag`/`@Operation`

### Guardrails

- Never run `DROP TABLE`, `DROP DATABASE`, `TRUNCATE`, or unscoped `DELETE FROM`.
- DDL schema changes require explicit user approval.
- Never delete `.env` or `application-prod.yml` without confirmation.

## Admin (Latinhouse.Admin)

### Commands
Run from `Latinhouse.Admin/`:
```bash
npm run dev      # start dev server
npm run build    # production build
npm run lint     # eslint
```

Stack: Next.js 16.2.7, React 19.2.4, TypeScript, Tailwind CSS 4.

**Important**: This Next.js version has breaking changes vs older versions/training data. Before writing code, check `node_modules/next/dist/docs/` for the relevant guide and heed deprecation notices.

### Structure
- `src/app/api/[...path]/route.ts`: catch-all proxy that forwards `/api/*` requests to the backend (`NEXT_PUBLIC_API_URL`, default `http://localhost:8080`).
- `src/app/{profiles,lessons,orders,coupons}/`: admin CRUD pages, one folder per domain matching `Latinhouse.Be` domains.
- `src/components/Sidebar.tsx`: nav shared across the admin layout.

## Frontend (Latinhouse.Fe)

Currently an empty scaffold (README only). Admin functionality previously here has moved to `Latinhouse.Admin/` — this project is reserved for a future end-user-facing frontend.

## Spec Kit Workflow

Features are developed via Spec Kit slash commands in sequence: `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement` (or `/speckit-full` to run the whole pipeline). Each feature lives under `specs/<NNN-name>/` with `spec.md`, `plan.md`, `tasks.md`. The project constitution (architecture rules, guardrails, quality gates above) is derived from `.specify/memory/constitution.md`.
