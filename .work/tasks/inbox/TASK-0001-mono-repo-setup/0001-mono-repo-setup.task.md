---
task_id: "0001"
created_by: "@cblank"
created_date: "2025-10-18"
schema_version: "1.0"
task_name: "Monorepo skeleton & DX bootstrap"
stage: "inbox"
priority: null
effort_estimate: null
tags: ["infrastructure", "dx", "bazel"]
current_dri: "@cblank"
status: "active"
blocked_by: null
paused_reason: null
depends_on: []
blocks: []
related_to: []
split_from: null
split_into: []
---

## Goal

Give us a clean, reproducible workspace with **bounded contexts**, **schema/codegen pipeline**, **boundary enforcement**, and **one-shot dev commands**, without slowing the v0 build loop.

## Inputs Required

* Bazel brief (bounded monorepo pattern)
* v0 repo layout (apps/, libs/, db/, infra/)
* Tooling: buf, openapi-generator, Docker, Make

## Implementation Details

1. **Decide scope for Bazel in v0**
   * Use Bazel for: component boundaries, graphs, CI gate on contracts, light test targets
   * Do not use Bazel (yet) for: Docker image builds, Next.js build, running containers. Keep `docker compose`

2. **Map components → Bazel types**
   * **platform/**: shared, foundational stuff (schemas, obs helpers)
   * **libraries/**: language-specific shared libs (generated protos/clients, otel wrappers)
   * **products/**: runnable services/apps (ui-web, gateway-rust, ai-gateway-py, orchestrator, consumers)
   * **data/**: migrations, seeds, configs (read as files only)
   * **tooling/**: scripts, boundary checker, schema generators
   * **tests/**: workspace tests

3. **Scaffold workspace**
   * Create WORKSPACE(.bazel), .bazelrc, .bazelversion
   * Add .workspace/ (types, rules, templates), BUILD.bazel stubs
   * Add Makefile with `make setup`, `make schemas`, `make check-boundaries`, `make graph`

4. **Contracts + codegen integration**
   * Place Protobuf & OpenAPI sources under platform/schemas/
   * Add Make target to run buf + openapi generator and write generated code into libs/*-protos and libs/ts-client (committed for v0)
   * Add CI check that regenerated code is up-to-date

5. **Boundary enforcement**
   * Implement boundary_rules.bzl aspect; set default visibility private; allowlists per type
   * Add one deliberate violation test to prove the gate works

6. **DX: pre-commit & CI**
   * Pre-commit runs: fmt/lint, make schemas, make check-boundaries
   * CI runs the same, fails on drift or boundary violations

7. **Documentation**
   * Root ARCHITECTURE.md (short)
   * LLM-CONTEXT.md template; one per component
   * DECISIONS.md (Bazel bounded scope now; deeper later)

## Expected Outputs

* Repo skeleton with Bazel, Make, and templates
* `make schemas` produces generated code; `make check-boundaries` enforces rules
* Dependency graph image via `make graph`
* Minimal example components build in Bazel (no container build)

## Acceptance Criteria

- [ ] `bazel build //...` completes with no violations (strict private visibility everywhere except explicit allowlists)
- [ ] `make schemas` regenerates code and CI fails if not committed
- [ ] `make check-boundaries` fails when a sample product→product dep is added, passes when removed
- [ ] `make graph` emits `workspace-deps.png`
- [ ] New component scaffolding works: `make new-component type=library name=obs-core`
