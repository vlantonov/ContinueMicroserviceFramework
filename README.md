# /continue — TDD Framework for Microservices (Claude Code)

A prompt framework for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that enforces strict TDD, Clean Architecture, and DDD for backend microservices in **C++, Python, or Go**. You type `/continue` and Claude executes the next atomic work unit — write a failing test, implement, refactor, commit — then stops.

Adapted from a production-app framework, narrowed to the backend-microservice case: one service per repo, no frontend, no browser tests.

## Why

LLMs write code that compiles but doesn't necessarily work, and without structure you re-explain the same constraints every conversation. This framework pins three things down:

- **Strict red-green-refactor.** Every line of production code is preceded by a failing test. The red phase predicts the exact error type and message, runs the test, and verifies the prediction field-by-field before accepting the red state.
- **Quality gates wired into the loop.** Each work unit runs `test-review` (assertion audit), coverage analysis with gap classification, and `refactor` (structural scan) before the commit. Not optional.
- **`progress.md` as persistent memory.** One checklist per story tracks exactly where work stopped. Context resets are safe — the file plus any ADRs in the story folder have everything needed to resume.

## How it feels

```
You: /continue 5
Claude: > Dispatching red-agent. Live progress: tail -f infrastructure/agent-progress.log
        [red-agent → test-review-agent → refactor-agent → commit]
        Step 14/47 done. Next: green-usecase.

You: /continue
Claude: [picks up where it left off]
```

`/continue` is the only command you type for day-to-day work. It reads `progress.md`, finds the next `[ ]` step, loads any relevant ADR, dispatches the right sub-agent (`red-agent`, `green-agent`, `refactor-agent`, `coverage-agent`, `test-review-agent`) in context isolation, runs the mandatory gates, commits, and stops. Sub-agents stream milestones to `infrastructure/agent-progress.log` — `tail -f` it in another terminal to watch RED → PREDICT → RUN → PASS unfold live.

Every feature follows the same pipeline:

```
interview → story → api-spec → test-spec
  → backend scenarios: red-acceptance → design → red-usecase → green-usecase
      → adapters-discovery → red-adapter(s) → green-adapter(s) → green-acceptance
  → integration / security / load / infrastructure scenarios
```

`adapters-discovery` is a gate: after `green-usecase`, `/continue` reads the usecase constructor, maps each port to an adapter, and rewrites `progress.md` with the concrete adapter steps before proceeding. When `/design-preview` surfaces a non-obvious decision, it produces an ADR in `decisions/NNN-slug-decision.md` under the story folder; subsequent `/continue` runs load it automatically.

## Architecture

Clean Architecture, dependency flow strictly inward:

```
backend/domain  ←  backend/usecase  ←  backend/adapters/{http,db,messaging,...}  ←  backend/application
acceptance/     (black-box HTTP / RPC tests, top-level)
```

Rules prohibit importing from outer layers in inner layers. Domain has zero framework dependencies. Adapters implement ports defined in usecase.

## Tech profiles

The framework is tech-agnostic across the supported microservice stacks. Set the `backend` key in `ProductSpecification/technology.md`:

| Concern | Available profiles |
|---------|-------------------|
| Backend | `cpp-cmake` · `python-django` · `go-stdlib` |

Profiles are independent. Each lives in `.claude/tech/{profile}/` with `coding.md`, `tdd.md`, `infrastructure.md`, and code templates. Adding a new profile (Rust, .NET, Node, JVM, etc.) means adding that directory; rules, agents, and skills resolve bindings dynamically through the single `backend` concern.

## Quick start

```bash
# 1. Copy the framework into your project
cp -r continue-microservice-framework/.claude continue-microservice-framework/CLAUDE.md continue-microservice-framework/ProductSpecification your-project/
cd your-project

# 2. Pick your stack in ProductSpecification/technology.md
#    tech-profile:
#      backend: go-stdlib

# 3. Start working
claude
> /continue 1
```

First run triggers the spec phase (`/interview` → `/story` → `/api-spec` → `/test-spec`), one skill at a time so you can review each. After the spec lands, every subsequent `/continue` executes one TDD work unit.

## What's inside

```
.claude/
├── rules/      universal principles (Clean Architecture, TDD, DDD, workflow)
├── agents/     red, green, refactor, coverage, test-review, test-runner, prompt-refactor
├── skills/     slash commands — /continue, /refactor, /test-coverage, /design-preview, ...
├── templates/  refactoring patterns, checklists, code scaffolds (tech-agnostic)
└── tech/       cpp-cmake/, python-django/, go-stdlib/ — pluggable backend profiles
```

Every AI decision traces back to a specific rule, checklist item, or template. Start with `CLAUDE.md` and `.claude/rules/workflow.md` to understand the loop; `.claude/skills/continue/SKILL.md` is the dispatcher.

## Limitations

- **Claude Code only.** Relies on subagents, slash commands, hooks, and tool use. Won't work with other AI coding tools without adaptation.
- **Opinionated.** Clean Architecture + DDD + strict TDD. If your project doesn't follow this structure, the framework will fight you.
- **Microservice scope only.** No frontend, no browser tests, no mobile, no monolith assumptions. The frontend layers and tooling from the original framework have been removed.
- **Context budget.** Rules load every conversation. On smaller context windows this leaves less room for code.

## References
* [Как я заставил ИИ писать код по книжке: Clean Architecture + TDD на автопилоте](https://habr.com/ru/articles/1023998/)
* <https://github.com/rakovi4/continue-framework>
* <https://github.com/rakovi4/continue-example>

## Acknowledgments

Draws on [Extreme Programming](http://www.extremeprogramming.org/) (Beck), [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) (Martin), [DDD](https://www.domainlanguage.com/ddd/) (Evans), and [Refactoring](https://refactoring.com/) (Fowler).

MIT License.
