# Test Evidence

## Recorded result

The final local branch was tested on upstream base commit:

```text
312b62ac95335e1762b70ceb8910374965bd2785
```

The final prototype head was:

```text
e1b0675477d5ec6144c56291031cd9eb4319a3a9
```

All 418 targeted tests passed:

| Test suite | Passed | Failed |
| --- | ---: | ---: |
| `codex-memories-write` | 59 | 0 |
| `codex-state` | 187 | 0 |
| `codex-api` | 172 | 0 |
| Total | 418 | 0 |

Repository checks also passed:

- targeted `just fix` for the three changed crates;
- `just fmt`;
- `git diff --check`;
- `just bazel-lock-update`;
- `just bazel-lock-check`.

## Behaviors covered

### Chunk construction

- Complete interaction turns remain together when they fit.
- The previous turn is marked as context only.
- A correction in the middle survives chunk construction.
- Dense JSON and Unicode use the same conservative byte based estimate as the runtime.
- Legacy rollouts receive deterministic synthetic turn identifiers.

### Oversized items and privacy

- A single message larger than 48,000 estimated tokens is split without silent loss.
- Oversized function arguments and custom tool input are split.
- Empty argument calls are not dropped.
- Large textual tool output is segmented.
- Binary, base64, media, and encrypted argument payloads become safe receipts or are removed.
- Signed media query data is not copied into model input.

### Retry and overflow

- Retryable provider errors retry only the failed chunk.
- HTTP and streamed context overflow share the same typed overflow path.
- Context overflow produces smaller child chunks.
- The unchanged overflowing parent is not resent.
- Request and leaf limits create visible failed coverage.

### Restart and ownership

- Completed chunks survive state database close and reopen.
- Matching checkpoints are reused after restart.
- Stale overflow children and descendants are removed.
- Ownership heartbeat prevents lease expiry during long model streams.
- Lost ownership cancels continued mutation.

### Finalization

- Partial or failed coverage cannot finalize Stage 1 as complete.
- Partial coverage does not enter Phase 2.
- Complete multi chunk output enters the existing Phase 2 input path.
- Existing small rollout output remains compatible.

## Synthetic end to end case

The end to end fixture used a 200,000 byte, two turn conversation. The second turn corrected an earlier decision.

Observed behavior:

1. The rollout produced two bounded Stage 1 requests.
2. The middle correction remained in the second request.
3. The previous turn appeared as context only.
4. Both Stage 1 outputs reached the existing Phase 2 input record.

## Exact proof limits

The tests use synthetic conversations, mock model streams, and temporary Codex homes.

They do not prove:

- behavior on a real user's rollout history;
- migration behavior on a real user's memory database;
- live provider model quality;
- exact tokenizer accounting;
- production cost, latency, throughput, or rate limit behavior;
- installed Codex desktop behavior;
- semantic memory precision or recall;
- semantic deduplication, contradiction resolution, confidence, or long term ranking.

The test result supports the local reliability design and its state transitions. It is not a production benchmark.

