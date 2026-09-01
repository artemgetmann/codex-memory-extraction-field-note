# Test Evidence

## Recorded result

The final local branch was tested on upstream base commit:

```text
312b62ac95335e1762b70ceb8910374965bd2785
```

The final local evaluation head was:

```text
27b30aca9
```

All 421 active targeted tests passed:

| Test suite | Passed | Failed |
| --- | ---: | ---: |
| `codex-memories-write` | 62 | 0 |
| `codex-state` | 187 | 0 |
| `codex-api` | 172 | 0 |
| Total | 421 | 0 |

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

## Historical structural replay

Two ignored, operator-invoked tests replayed naturally occurring local rollout shapes without a
provider call or production memory database. They verified complete source-turn coverage, bounded
chunk construction, checkpoint persistence, and resume after a forced failure. Transcript content
was not printed, committed, or uploaded by these tests.

A third ignored test exported the redacted baseline and candidate prompts for explicit operator
review. It did not call a model.

## Private live model comparison

The follow-up quality check used three redacted historical technical rollouts. Ground truth was
written before generation and contained 28 facts covering the head, middle, and tail. Critical facts
counted twice.

Protocol:

- current head and tail Stage 1 prompt versus turn aligned bounded chunk prompts;
- `gpt-5.6-luna` with low reasoning for every request;
- 16 schema constrained calls, with zero retries and zero invalid outputs;
- anonymous A and B scoring before the arm mapping was opened;
- no production memory database and no Phase 2 call.

Observed result:

| Measure | Baseline | Bounded candidate |
| --- | ---: | ---: |
| Declared facts recovered | 28 of 28 | 28 of 28 |
| Weighted fact score | 43 of 43 | 43 of 43 |
| Weighted middle fact score | 21 of 21 | 21 of 21 |
| Blinded usefulness wins | 3 of 3 | 0 of 3 |
| Combined output characters | 28,516 | 101,559 |

The candidate produced no recall win and was 3.56 times longer. Its deterministic merge preserved
repeated intermediate states, including one rejected instruction beside its later correction. The
predeclared stop rule therefore ended the experiment before Phase 2.

Actual live usage was 2,098,105 input tokens, 47,360 cached input tokens, 33,109 output tokens, and
1,750 reasoning output tokens across 16 completed calls. The input total includes the surrounding
`codex exec` runtime instructions and is therefore larger than the prompt only estimate.

## Exact proof limits

The reliability tests use synthetic conversations, mock model streams, and temporary Codex homes.
The historical structural replay and private quality comparison close part of that ecological gap,
but the sample is deliberately small.

They do not prove:

- migration behavior on a real user's memory database;
- general live provider model quality beyond three redacted conversations and one model;
- exact tokenizer accounting;
- production cost, latency, throughput, or rate limit behavior;
- installed Codex desktop behavior;
- general semantic memory precision or recall beyond this small sample;
- semantic deduplication, contradiction resolution, confidence, or long term ranking.

The test result supports the local reliability design and its state transitions. It is not a production benchmark.
