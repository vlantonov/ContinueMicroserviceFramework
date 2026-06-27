# CLAUDE.md

TDD Framework Template — Clean Architecture backend microservice (multi-module) + black-box acceptance tests.

Technology stack and commands are declared in `ProductSpecification/technology.md`. Read that file (and the selected profile under `.claude/tech/{backend}/`) for build commands, test commands, and framework-specific conventions.

## Architecture

```
application → adapters → usecase → domain
```

All backend modules live under `backend/`. Never place them in the project root.

| Module | Description | Dependencies |
|--------|-------------|--------------|
| `backend/domain` | Entities, value objects, exceptions | None (code generation only) |
| `backend/usecase` | Application services, port interfaces | domain |
| `backend/adapters/{http,rest,grpc}` | Inbound network controllers / handlers | usecase |
| `backend/adapters/{db,h2,sqlite,postgres}` | Database repositories, migrations | usecase |
| `backend/adapters/email` | Mail integration | usecase |
| `backend/application` | Entry point, wiring | all modules |
| `acceptance` | Black-box API tests (top-level) | None (HTTP / RPC only) |

The specific adapter directories depend on the selected tech profile — inspect `.claude/tech/{backend}/templates/` for the templates each profile provides. Dependency flow is strictly inward. Never import from outer layers in inner layers.

## Scope

This is a **microservice-only** framework. There is no frontend, no browser-based testing, and no UI mockup workflow. Story specs and tests cover API, integration, security, load, and infrastructure concerns only.

## Interaction Rules

- **Never block longer than 30 seconds.** No `sleep 60`, no `TaskOutput` with 5-minute timeouts. Use `run_in_background: true` for long commands, then poll with short separate calls (≤30s each) so the user sees progress between each check.
