# How We Win Speed Demon 2026

## Winning condition

Win the Overall Championship by posting valid scores on at least three tracks and maximizing the geometric mean of our best three totals. Do not sacrifice two strong tracks chasing a speculative fourth: a single weak score pulls the geometric mean down.

Our operating principle is **correctness first, then the largest measured improvement per engineering hour**. A zero from a hash mismatch, sandbox violation, or missed time cap is worth nothing.

## Recommended portfolio

1. **Track C — Join:** make this the first secure score. It has the smallest reference runtime, a narrow task, and its main risk (hostile CSV correctness) is easy to test systematically.
2. **Track A — Ingest:** make this the second secure score. Fixed-format parsing and a limited endpoint universe make it a strong Rust/mmap target.
3. **Track D — Traverse:** make this the third secure score. CSR plus per-thread Dijkstra state is a clear, high-upside implementation path.
4. **Track B — Index:** pursue only after A/C/D are valid, or if early profiling shows unusually high gains. It has the greatest architectural risk because indexing and query semantics must both be perfect.

This portfolio gives a championship-eligible result without betting everything on the most complex track. If all four become viable, submit all four and let the best three count.

## Day 0: establish the baseline

Before writing optimized code:

- Audit the official material currently available: the frozen spec, Python reference, generator, complete local harness, starter/harness README, image allowlist, published clarifications, kickoff transcript, and (after kickoff) full data plus expected hashes. Record paths, versions, and missing items in the track’s `PERF.md`; do not assume the README summary overrides the official spec.
- Clone the official specification/reference repository outside this repo.
- Read `CONVENTIONS.md`, the selected track spec, and the Python reference line by line.
- Generate sample data for seeds 1 through 5.
- Run every reference implementation and save its output hashes, wall time, peak RSS, output formatting rules, sort order, rounding behavior, and failure cases.
- Make a minimal Rust submission for each chosen track: exact Dockerfile, zero dependencies, correct `/data` discovery, exactly one `/out` file, exit code 0.
- Verify the Docker build works without network access before spending time on algorithms.

Create a one-page per-track checklist for: input filenames, output filename, integer/float formatting, ordering, ties, malformed-input handling, and empty cases. Treat that list as the contract.

## The delivery sequence

### Phase 1 — secure valid submissions

Implement a simple, readable Rust version for C, A, and D in that order. It may be slower than the intended final design, but it must pass every generated sample and meet the practice cap.

For every implementation change:

1. Run the reference and our solution on the same seed.
2. Compare SHA-256 hashes, not a visual diff alone.
3. Test seeds 1–5 plus deliberately tiny hand-crafted edge cases.
4. Run the official `check.py` in the same Docker-shaped environment.
5. Record time, peak memory, LOC, and the hypothesis in that track’s `PERF.md`.

Never combine a large refactor with an unverified performance trick. Keep a known-good commit or copy outside the `track-*` folder so a regression is cheap to unwind.

### Phase 2 — make the big gains

#### Track C: hostile CSV join

- Build the lookup into a flat array or compact hash table.
- Scan bytes once; only recognize a record boundary while outside quoted fields.
- Correctly handle `""`, embedded newlines, both line endings, and any UTF-8 BOM occurrence mandated by the spec.
- Extract only the columns needed for the join and aggregate; do not allocate every field or every row.
- Match the reference’s numeric accumulator and final sort tie-breakers exactly.

The key test is a fuzz corpus of tiny CSV inputs combining quotes, commas, CRLF/LF, BOMs, empty fields, negative/large amounts, and newlines inside notes.

#### Track A: fixed-format ingestion

- Read input with read-only `mmap`; split it into newline-aligned ranges for worker threads.
- Parse fixed offsets directly from bytes: hour, endpoint, status, and latency.
- Give each worker a local fixed-size table and local aggregation state, then merge deterministically.
- Avoid per-line `String`, regex, hash-map churn, and full-file copies.
- Use the reference’s exact quantile index rule. Confirm it on data sets with 0, 1, 2, and duplicate observations.

Do correctness work before parallelism: first prove a single-thread byte parser, then partition lines and merge.

#### Track D: graph traversal

- Parse edges into compact primitive arrays and construct CSR (`offsets`, `neighbors`, `weights`).
- Use a binary heap with stale-entry skipping unless profiling proves a more specialized queue earns its complexity.
- Parallelize independent queries with an atomic work counter.
- Reuse each worker’s distance/visited buffers and use epochs to avoid clearing arrays per query.
- Preserve result order by writing results to indexed slots and emitting only after all workers finish.

Profile whether graph parsing or the queries dominate. Optimize the dominant half first; avoid an elaborate parallel parser if 2,000 searches consume the time.

#### Track B: only after eligibility is secured

- Parse all queries first and collect the vocabulary actually needed.
- Distinguish Boolean-only terms from phrase terms.
- During corpus scanning, discard irrelevant terms; retain positions only when phrase matching needs them.
- Validate Boolean precedence, negation universe, tokenization, document ordering, phrase boundaries, and duplicate behavior against the reference.

This is the highest memory opportunity, but also the easiest place to make a quiet semantic mistake. Its optimization should not delay A/C/D validity.

## Score optimization rules

The efficiency coefficient heavily favors time, then memory:

`K = (Tref/T)^0.50 × (Mref/M)^0.20 × (Sref/S)^0.15 × (Dref/D)^0.15`

- **Time:** optimize hot parsing/scanning loops, data layout, and parallel independent work. Measure five runs; use the middle three as the board does.
- **Memory:** aim for streaming, mmap, local fixed storage, and primitive arrays. Memory savings cap at 50×, so do not trade away huge runtime wins merely to reduce already-small memory.
- **LOC:** source size cannot improve the score below the reference, but excess LOC hurts. Keep production source compact; store tests and generators outside `track-*`.
- **Dependencies:** stay at zero. A crate adds score penalty and offline-build risk.
- **Review:** use descriptive but economical names, clear phases, and a concise `PERF.md`. Human review is 35% of each track’s score.

## Board discipline

The board permits one run every 15 minutes, so every push must answer one question: “Did this specific change improve a valid solution?”

Before a push, require all of the following:

- The official-materials inventory was rechecked for newly issued specs, hashes, harness changes, or clarifications.
- `cargo build --release --offline` succeeds from the track directory.
- The Dockerfile uses exact allowlisted digests and has no forbidden feature.
- No network-dependent build step or external dependency exists.
- Exactly one output file is produced under `/out`.
- Hashes match the reference on all local seeds.
- `scc track-x` is known and no test/dead files inflate it.
- `PERF.md` states the change, expected effect, local measurement, and board run ID once available.

Keep a small board log with timestamp, commit, track, hypothesis, correctness result, T, M, LOC, K, and next action. Do not push “maybe faster” changes without local measurements.

## Final 12-hour checklist

1. Freeze the best valid A, C, and D candidates.
2. Regenerate full data using several unseen seeds and compare hashes.
3. Rebuild each image from scratch with network disabled.
4. Re-run five local benchmarks per track and calculate approximate K.
5. Choose only improvements with clear projected K upside and low correctness risk.
6. Clean each `track-*` directory: retain only required source, Cargo files, Dockerfile, `.speed-demon.toml`, and `PERF.md`.
7. Confirm inactive track folders are deleted if the event rules require that for scoring.
8. Make the final board submissions early enough to recover from a build or hash failure.

## Decision rules under pressure

- If a track is not hash-correct, stop optimizing it and fix correctness.
- If a valid track is within the cap, bank it before attempting a risky rewrite.
- If an optimization saves less than 5% locally, prefer readability or move to the next bottleneck.
- If a change increases LOC or dependencies, demand a measured K improvement large enough to justify it.
- If two tracks are strong and one is merely valid, focus on raising the weakest of the three: geometric mean rewards balanced scores.

The practical route to victory is: bank C, bank A, bank D, measure relentlessly, then spend remaining time on the bottleneck that most improves the weakest championship-counting score.
