# Gliding Horse Agent OS — Agent Instructions

## Project Identity

- **Name**: glidinghorse
- **Type**: Rust workspace — AI Agent Operating System
- **Workspace members**: `crates/hyperspace-engine`, `crates/ontologies`, `apps/gliding_code`

## Build Commands

```bash
cargo build --release           # Full release build
cargo build -p code_cli        # Gliding Code binary only
cargo test                     # Test suite
cargo clippy -- -D warnings     # Lint
cargo fmt --check              # Format check
cargo run --release --example readme_performance   # Performance benchmarks
```

## gRPC Proto Compilation

`build.rs` compiles two proto files:
- `proto/pdca_core.proto`
- `apps/software_engineering_single/proto/se_app.proto`

Proto changes trigger rebuilds via `cargo:rerun-if-changed` directives.

## Key Crates

| Crate | Role |
|-------|------|
| `crates/hyperspace-engine` | HNSW vector store, WAL, Poincaré embeddings |
| `crates/ontologies` | SHACL, reasoner, lint, diff (feature-gated) |
| `apps/gliding_code` | ratatui TUI CLI |

> **Note**: `apps/software_engineering_team` and `apps/software_engineering_single` are
> standalone Go/TS projects outside the Rust workspace. They are not built by `cargo`
> commands above and have their own build tooling.

## Feature Flags

| Flag | Effect |
|------|--------|
| `default` | Enables `ontology` |
| `ontology` | SHACL, reasoner, lint, diff |
| `ontology-embeddings` | Adds ONNX model (~100MB) |
| `ontology-causal` | Requires `python3` + `dowhy` at runtime (⚠️ `python3` unavailable on this Windows host — feature may not work) |
| `live-tests` | Enables real-provider E2E tests (disabled by default) |

## Module Layout

```
src/
├── api/           gRPC/REST gateway
├── batch/         Background batch agents
├── causal/        Causal inference engine (dowhy integration)
├── config/        Agent roles, templates, rules
├── core/          SA scheduler, AgentRunner, PDCA orchestration
├── gateway/       SyscallGate, ToolGuard, StageGate
├── graph_features/  Graph feature extraction and similarity
├── jsonld/        Semantic engine (context, framing, type routing)
├── knowledge_graph/  Knowledge graph management
├── llm/           SSE streaming, response parsing
├── memory/        L0–L3 memory, MESI consistency, prefetch
├── methodology/   5W2H, Jikotei Kanketsu, TPS
├── perception/    Situation, health, pattern, conflict engines
├── root_cause/    RootCauseEngine
├── skill_graph/   Store, discovery, evolution, conflict detection
├── snapshots/     Snapshot management
├── templates/     Prompt templates and schemas
├── tools/         ToolExecutor, MCP, Hooks, result router
├── utils/         Shared utility functions
└── worker/        Task workers, sandbox execution
```

## Important Conventions

- **Error handling**: `anyhow` for application code, custom `thiserror` types for library crates
- **Async**: All I/O and tool calls are async, Tokio runtime
- **JSON-LD**: Use `@id`, `@type`, `@context` consistently; register all new IRIs in ontology
- **Commits**: [Conventional Commits](https://www.conventionalcommits.org/): `feat(skill_graph):`, `fix(memory):`, etc.
- **Tests**: Unit tests inline (`#[cfg(test)]`); integration tests in `tests/`

## CI Checks

GitHub Actions (`.github/workflows/`):
- `core-quality.yml` — clippy, fmt, unit tests
- `namespace-consistency.yml` — ontology IRI consistency
- `release-gliding-code.yml` — cross-platform binary builds

## Environment Variables

See `.env.example`. Key prefix: `AGENT_OS_` (gateway URL, API key, model, output dir, RUST_LOG).

## Development

- **Rust version**: 1.98+ (workspace requirement; confirmed rustc 1.98.0)
- **Recommended IDE**: VS Code + rust-analyzer
- **Proto files**: create at `proto/your.proto`, then add to `build.rs` AND update the
  gRPC Proto Compilation section above. Both files must be kept in sync; rebuilds
  are triggered automatically via `cargo:rerun-if-changed` directives
- **No new batch agent**: add template to `templates/prompts/batch/` + config entry + handler in `src/batch/handlers.rs`

## Useful Commands

```bash
# Run specific integration test
cargo test -p glidinghorse --test test_e2e

# Run with live tests (requires API keys)
cargo test --features live-tests

# Inspect ontologies namespace consistency
bash scripts/check_namespace.sh

# Check version consistency across crates
bash scripts/check_version_consistency.sh

# Test MCP integration
bash scripts/test_mcp.sh
```
