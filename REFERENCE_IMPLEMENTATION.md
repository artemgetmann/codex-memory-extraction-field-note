# Reference Implementation

## Status

This is a local reference implementation based on public Codex source. It was never pushed, opened as a pull request, or represented as OpenAI production code.

Upstream base:

```text
312b62ac95335e1762b70ceb8910374965bd2785
```

Local evaluation head:

```text
27b30aca9
```

Diff size:

```text
23 files changed
4,541 insertions
86 deletions
```

## Commit structure

| Commit | Purpose |
| --- | --- |
| `a0dac8803` | Classify HTTP context overflow separately from generic invalid input. |
| `e62cf06a4` | Persist chunk plans, checkpoints, leases, and truthful coverage. |
| `d9ee548b3` | Build bounded turn aligned chunks and sanitize oversized items. |
| `c75230fdd` | Run checkpointed extraction with adaptive overflow splitting. |
| `132a2009b` | Cover the complete Stage 1 lifecycle with synthetic startup tests. |
| `e1b067547` | Close empty plan, coverage reporting, and request cap gaps. |
| `98c6feb8c` | Replay historical rollout shapes for coverage and restart behavior. |
| `27b30aca9` | Export redacted baseline and bounded prompts for blinded quality testing. |

## Changed areas

### API error classification

```text
codex-rs/codex-api/src/api_bridge.rs
codex-rs/codex-api/src/api_bridge_tests.rs
```

Maps `context_length_exceeded` HTTP responses to the existing context overflow error type so Stage 1 can shrink the input instead of treating it as generic invalid input.

### Chunk planning and execution

```text
codex-rs/memories/write/src/stage1_chunks.rs
codex-rs/memories/write/src/stage1_runner.rs
codex-rs/memories/write/src/phase1.rs
codex-rs/memories/write/src/prompts.rs
```

Builds complete turn groups, handles oversized items, renders bounded prompts, checkpoints results, adapts to overflow, renews ownership, and finalizes only complete coverage.

### Persistence

```text
codex-rs/state/memory_migrations/0002_stage1_chunk_outputs.sql
codex-rs/state/src/model/memories.rs
codex-rs/state/src/runtime/memory_chunks.rs
codex-rs/state/src/runtime/memories.rs
```

Stores chunk plans, source digests, status, outputs, overflow ancestry, and coverage. Ownership fenced APIs protect concurrent startup workers.

### Tests

```text
codex-rs/memories/write/src/stage1_chunks_tests.rs
codex-rs/memories/write/src/startup_tests.rs
codex-rs/memories/write/src/startup_tests/historical_rollout_tests.rs
codex-rs/memories/write/src/startup_tests/memory_quality_eval.rs
codex-rs/state/src/runtime/memory_chunks_tests.rs
codex-rs/codex-api/src/api_bridge_tests.rs
```

Covers planner behavior, synthetic startup flow, restart persistence, failure visibility, ownership, and overflow classification.

## Why patches are not included initially

The Codex contribution guide does not accept external code contributions or pull requests. Publishing a 4,541 line patch dump would make this field note harder to inspect and could look like an attempt to route around that policy.

The initial artifact therefore includes:

- exact base and prototype commit identities;
- a complete changed path map;
- architecture and state contracts;
- test counts and test scenarios;
- exact proof limitations.

Sanitized patches remain available locally. If a Codex maintainer asks for them, publish them as a versioned appendix pinned to the exact upstream base and carrying the required Apache 2.0 attribution and notices.
