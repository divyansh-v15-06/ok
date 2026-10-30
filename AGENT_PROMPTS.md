# Speed Demon 2026 — Track Agent Prompts

Give one of the prompts below to a separate coding agent working in this repository. Assign **one track per agent**. The agent must read the repository documents named in its prompt before editing anything.

## Coordination contract for every agent

- Work only in your assigned `track-<x>/` directory. Do not edit another track, root documentation, or shared configuration.
- The source tree may be worked on by other agents. Preserve unrelated changes and do not reset, delete, or reformat their work.
- The event requires one independent submission per track. All code needed by your track must live inside its track directory; do not depend on sibling directories at runtime or build time.
- Implement and verify the work rather than only proposing code. Report exact files changed, checks run, hash outcomes, timings/memory if available, and anything still unverified.
- Do not use external runtime dependencies. Build must function offline.
- Favor a correct, cap-compliant baseline before risky optimization. Never claim correctness without a byte-for-byte hash comparison against the reference output.
- Before coding, audit what the organizers have actually supplied: spec, reference, generator, complete harness/`check.py`, starter and harness READMEs, Docker/image allowlist, published clarifications, kickoff transcript, full public data, and expected output hash. List the local path/version of each item in your first `PERF.md` entry. If an item is absent, inspect the official competition site/event channels; do not make up its behavior.
- Recheck that inventory before every board push. Official specs and clarifications win over any summary in this repository.

---

## Prompt: Track A — Ingest

```text
You own Track A (Ingest) in the Speed Demon 2026 repository. Your responsibility is to create and finish a competitive, correct submission in `track-a/`.

Start by reading these repository documents completely:
1. `README.md`
2. `WIN_WALKTHROUGH.md`

Then inspect the official materials outside the submission repo, as described in the README:
- `../speed-demon-2026/CONVENTIONS.md`
- `../speed-demon-2026/specs/a-ingest.md`
- `../speed-demon-2026/reference/a_ingest.py`
- `../speed-demon-2026/generators/a_ingest.py`

Also locate the supplied harness/`check.py`, starter/harness README, current image allowlist, official clarifications, the kickoff transcript if published, and the full public data plus expected output hash. Record where each was found before implementation.

Track objective:
- Read the Track A input from `/data` and write exactly `/out/summary.txt`.
- The output SHA-256 must match the Python reference byte-for-byte.
- The full data set is approximately 239 MB / 3.1M fixed-format space-delimited log lines. The board cap is 96.15 seconds and the final cap is 321 seconds.
- Produce a zero-dependency Rust submission with an allowlisted, offline-capable Docker build and the required metadata files.

Implementation plan:
1. Learn exact input filenames, output format, sorting, quantile rank convention, error-rate convention, and all edge conditions directly from the spec/reference. Record them before coding.
2. Create a simple single-thread correct baseline first. Verify output hashes on samples and seeds 1–5.
3. Replace allocation-heavy parsing with byte-based parsing. Parse timestamp hour, endpoint, status, and latency directly from line bytes only after proving fixed offsets are actually valid under the spec.
4. Use read-only mmap or streaming only if it is supported by the standard library/OS interface without dependencies and remains correct in the sandbox. Partition input only on newline boundaries.
5. Use per-worker local aggregation and deterministic merging. Avoid global locks in the hot loop.
6. Implement percentiles using precisely the reference’s rank/index and interpolation behavior; test empty, one-element, two-element, duplicates, and boundary percentile cases.
7. Keep source compact: production code only inside `track-a/`; tests/generators outside that directory if the competition LOC tool counts the whole folder.

Required verification:
- `cargo build --release --offline`
- generate sample/full data when feasible, for seeds 1–5
- run reference and solution on identical data and compare SHA-256
- use `check.py a --data ...` where available
- run `scc track-a` and inspect source/dependency footprint
- verify Docker build and runtime restrictions as far as local tooling permits

Maintain `track-a/PERF.md` with the hypothesis, design, local performance/memory results, correctness evidence, and later board run IDs. Finish by reporting files changed, exact test commands/results, current limitations, and the next highest-value optimization only if correctness is already secured.
```

---

## Prompt: Track B — Index

```text
You own Track B (Index) in the Speed Demon 2026 repository. Your responsibility is to create and finish a competitive, correct submission in `track-b/`.

Start by reading these repository documents completely:
1. `README.md`
2. `WIN_WALKTHROUGH.md`

Then inspect the official materials outside the submission repo:
- `../speed-demon-2026/CONVENTIONS.md`
- `../speed-demon-2026/specs/b-index.md`
- `../speed-demon-2026/reference/b_index.py`
- `../speed-demon-2026/generators/b_index.py`

Also locate the supplied harness/`check.py`, starter/harness README, current image allowlist, official clarifications, the kickoff transcript if published, and the full public data plus expected output hash. Record where each was found before implementation.

Track objective:
- Build and query an inverted index over roughly 256 MB of corpus data and 50,000 Boolean/phrase queries.
- Read from `/data`, write exactly `/out/results.txt`, and match the reference SHA-256 byte-for-byte.
- Board/final caps are 169.05s / 564s. Use zero-dependency Rust and an offline Docker build.

This track has high upside but high semantic risk. Establish behavior from the reference before optimizing: tokenization, normalization, document IDs, query grammar and precedence, NOT universe, duplicates, phrase boundaries/positions, result ordering, formatting, and empty-result behavior.

Implementation plan:
1. Build the simplest correct index/query evaluator and hash-check it on hand-crafted corpus/query fixtures plus generated samples.
2. Parse all queries first. Derive the exact query vocabulary and identify which terms need positional postings for phrase searches.
3. On corpus scan, discard tokens not required by any query. For Boolean-only terms retain only the least expensive representation that preserves reference semantics; retain positions only for phrase terms.
4. Carefully separate parsing/index construction time from query execution only for profiling; both count in the official wall time.
5. Optimize allocation and data layout after hashes match. Use compact primitive postings structures and avoid per-token strings in hot paths where possible.
6. Add parallelism only for independent work and preserve deterministic output order.
7. Do not sacrifice query semantics for a memory optimization. Test nested Boolean expressions, NOT, repeated terms, phrase overlap, punctuation/case boundaries, and missing terms.

Required verification:
- `cargo build --release --offline`
- reference-versus-solution SHA-256 comparisons for generated data and seeds 1–5
- `check.py b --data ...` where available
- full behavior tests for query grammar and phrase positions
- `scc track-b`; confirm zero runtime dependencies and Docker viability

Maintain `track-b/PERF.md` with correctness evidence, storage design, measured time/RSS, LOC, and each optimization hypothesis. Finish by reporting files changed, tests/hashes, current measured bottleneck, and remaining risk.
```

---

## Prompt: Track C — Join

```text
You own Track C (Join) in the Speed Demon 2026 repository. Your responsibility is to create and finish a competitive, correct submission in `track-c/`.

Start by reading these repository documents completely:
1. `README.md`
2. `WIN_WALKTHROUGH.md`

Then inspect the official materials outside the submission repo:
- `../speed-demon-2026/CONVENTIONS.md`
- `../speed-demon-2026/specs/c-join.md`
- `../speed-demon-2026/reference/c_join.py`
- `../speed-demon-2026/generators/c_join.py`

Also locate the supplied harness/`check.py`, starter/harness README, current image allowlist, official clarifications, the kickoff transcript if published, and the full public data plus expected output hash. Record where each was found before implementation.

Track objective:
- Parse roughly 204 MB / 3.32M records of hostile CSV, join with the lookup input, and write exactly `/out/aggregates.txt`.
- Output must match the Python reference SHA-256. Board/final caps are 43.32s / 144s.
- Use dependency-free Rust and an offline-compatible Docker build.

Correctness risks are the priority: quoted commas, escaped quotes (`""`), embedded newlines in quoted fields, mixed LF/CRLF, and UTF-8 BOMs that may appear mid-file. Never split records simply on newline.

Implementation plan:
1. Extract the exact CSV and aggregation behavior from the reference: headers, malformed records, BOM treatment, numeric parsing/rounding, join misses, output sorting, and ties.
2. Build a small adversarial fixture suite first. Include commas/newlines inside quotes, escaped quotes, empty fields, CRLF/LF mixtures, BOM at start and mid-file, join misses, and numeric extremes.
3. Implement a one-pass byte state machine that recognizes row boundaries only outside quotes. Match the reference’s handling of any malformed input rather than inventing behavior.
4. Extract only fields that influence the join/aggregate; do not allocate all CSV cells or rows.
5. Load the lookup data into a compact direct-indexed or open-addressed representation according to the actual key domain.
6. Accumulate with the exact reference numeric type and output formatting. Sort exactly with every specified tie-breaker.
7. Optimize after exhaustive hash validation; the first target is allocation elimination and sequential scanning, not unsafe parallel CSV partitioning.

Required verification:
- `cargo build --release --offline`
- all adversarial fixture outputs compared byte-for-byte to the reference
- generated sample/full runs where feasible, seeds 1–5, SHA-256 checks
- `check.py c --data ...` where available
- `scc track-c` and local Docker checks

Maintain `track-c/PERF.md` with parser invariants, correctness cases, measurements, LOC, and change log. Finish by reporting changed files, commands/results, any untested reference behavior, and the next safe optimization.
```

---

## Prompt: Track D — Traverse

```text
You own Track D (Traverse) in the Speed Demon 2026 repository. Your responsibility is to create and finish a competitive, correct submission in `track-d/`.

Start by reading these repository documents completely:
1. `README.md`
2. `WIN_WALKTHROUGH.md`

Then inspect the official materials outside the submission repo:
- `../speed-demon-2026/CONVENTIONS.md`
- `../speed-demon-2026/specs/d-traverse.md`
- `../speed-demon-2026/reference/d_traverse.py`
- `../speed-demon-2026/generators/d_traverse.py`

Also locate the supplied harness/`check.py`, starter/harness README, current image allowlist, official clarifications, the kickoff transcript if published, and the full public data plus expected output hash. Record where each was found before implementation.

Track objective:
- Parse a 5M-edge directed weighted graph and answer 2,000 bounded-distance shortest-path queries.
- Read from `/data`, write exactly `/out/distances.txt`, and match the reference SHA-256 byte-for-byte.
- Board/final caps are 218.07s / 727s. Use dependency-free Rust and an offline-compatible Docker build.

Implementation plan:
1. Derive exact graph/query semantics: vertex numbering, duplicate edges, self-loops, weight range, unreachable representation, distance bounds, integer overflow rules, input/output ordering, and formatting.
2. Write a correct sequential baseline using the reference-compatible shortest-path algorithm. Hash-test tiny hand-made graphs first, then generated samples/seeds 1–5.
3. Store the graph in a compact CSR representation: offsets plus contiguous neighbor/weight arrays. Avoid adjacency `Vec<Vec<_>>` if memory/profile data confirms CSR is superior.
4. Profile parsing/build versus searches. Optimize whichever consumes most measured wall time.
5. Run independent queries across worker threads. Use an atomic query index or equivalent and store each answer at its original query position.
6. Allocate each worker’s distance and visit-epoch state once; use epoch counters to avoid O(V) clears while guarding epoch overflow according to the actual query count.
7. Use a correct priority queue implementation with stale-entry skipping, unless the spec proves a specialized bounded-weight method is valid. Do not introduce heuristics that can change exact distances.

Required verification:
- `cargo build --release --offline`
- hand-built graph cases covering unreachable nodes, equal routes, duplicates, zero/maximum weights if valid, and bounds
- reference-versus-solution SHA-256 checks for generated data, seeds 1–5
- `check.py d --data ...` where available
- `scc track-d`, zero-dependency check, and local Docker validation

Maintain `track-d/PERF.md` with CSR design, query-state design, correctness evidence, timing/RSS/LOC, and optimization log. Finish by reporting changed files, test/hash results, measured bottleneck, and remaining risks.
```
