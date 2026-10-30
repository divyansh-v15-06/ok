# CLAUDE.md — Speed Demon 2026 Agent Context & Systems Blueprint

## 1. Event Overview & Core Objective
You are an expert systems and performance-engineering agent competing in **Speed Demon 2026 ("Every Millisecond Is a Soul")**, a 72-hour performance hackathon (Oct 30, 15:00 UTC to Nov 2, 15:00 UTC).
- **The Task:** Replace deliberately slow, naive Python reference implementations across 4 tracks (`track-a` through `track-d`) with ultra-fast, low-memory implementations in any allowlisted language (recommended: zero-dependency Rust).
- **Single Repo Structure:** One repository per team on the `speed-demon` branch holds directories `track-a`, `track-b`, `track-c`, and `track-d`.
  - Delete any `track-*` directory we are not actively entering. Leaving a folder untouched causes the board to mark it `unchanged` and skip it.
  - Each track is built and scored strictly in isolation on its own folder. Shared code must be copied into each track folder that uses it and counts toward each folder's source size.
- **Championship Eligibility:** You must score on **at least 3 tracks** to qualify for the Overall Championship ($800 1st, $400 2nd, $200 3rd). Overall ranking is the **geometric mean** of your best 3 track totals. Individual track winners (highest $K$ per track) earn $50 each, plus $200 ($100 x 2) for the Write-Up Side Quest.

---

## 2. The Three Hard Gates (Fail Any = Score 0)
1. **Byte-for-Byte Correctness:** Read inputs from `/data` (read-only) and write **exactly one output file** to `/out`, then exit with code `0`. The output file's SHA-256 hash must match the Python reference output byte-for-byte.
2. **Time Caps:**
   - **Live Practice Board Cap:** Must finish within `3x` the reference runtime (`T_ref`).
   - **Final Holdout Cap:** Must finish within `10x` the reference runtime (`T_ref`).
3. **Air-Gapped Sandbox:** Zero network access (`--network=none`) during **both** `docker build` and container execution.

---

## 3. Scoring Formula & Strategic Optimization Rules
Each track is scored out of 100 points:
- **65% Automated Machine Score:** Derived from the Efficiency Coefficient (`K`).
- **35% Anonymized Human Panel Review:** 27% for code readability, structure, and engineering technique + **8% for the required `PERF.md` optimization log**.

### Efficiency Coefficient (`K`) & Speed Score
```text
K = (T_ref / T)^0.50 * (M_ref / M)^0.20 * (S_ref / S)^0.15 * (D_ref / D)^0.15
Speed Score = clamp(50 * log10(K), 0, 100)
```
*(Note: K = 1 -> 0 pts; K = 10 -> 50 pts; K = 100 -> 100 pts ceiling).*

### Mandatory Engineering Rules Dictated by `K`
1. **Wall Time (`T`, 50% weight, Uncapped):**
   - Measured on the scoring box as the mean of the middle 3 of 5 runs.
   - Parallelize across all 12 hardware threads (`6 P-cores`, turbo off).
2. **Peak Memory (`M`, 20% weight, Capped at 50x reduction):**
   - Read from the kernel high-water mark. Achieving `M_ref / M >= 50` gives a `50^0.20 = 2.1867x` multiplier to `K` (equivalent to running `4.78x` faster in wall time).
   - Use read-only `mmap` (`MADV_SEQUENTIAL`) or small fixed-size streaming buffers. Avoid loading entire datasets onto the heap.
3. **Source Size (`S`, 15% weight, Capped at 1.0x — Penalty Only):**
   - Measured in LOC by `scc` after canonical formatting across the track folder.
   - Writing shorter code than `S_ref` gives NO bonus (`max 1.0x`), but exceeding `S_ref` actively penalizes `K`.
   - Keep code in a single compact file (`src/main.rs`). Never commit unit test files, dead code, or verbose boilerplate inside `track-*/`.
4. **Dependency Weight (`D`, 15% weight, Capped at 1.0x — Penalty Only):**
   - Calculated as `D = 1 + direct_non_stdlib_runtime_dependencies`.
   - The Python reference uses stdlib only (`D_ref = 1`). Any external crate/library (`D > 1`) reduces `K` and complicates offline Docker builds.
   - **Rule:** Use `dependencies = 0` (`D = 1`). Rely strictly on the language standard library and OS syscalls.

---

## 4. Hardware Environment & Execution Contract
Every push to the `speed-demon` branch is built (on E-cores) and benchmarked (on P-cores) on a single dedicated server in Falkenstein (rate-limited to 1 run per team per 15 minutes):
- **CPU:** 6 P-cores / 12 threads, Turbo Boost OFF.
- **Memory:** 8 GB RAM maximum, 0 swap (exceeding 8 GB triggers OOM failure).
- **Process Limit:** 512 max processes/threads.
- **Scratch Space:** `/tmp` mounted as 1 GB `tmpfs`.
- **Filesystem:** Root `/` and `/data` are read-only; write **only** the single target output file to `/out`.
- **Environment Variable:** `TRACK` is set to `a`, `b`, `c`, or `d` at runtime.

---

## 5. Dockerfile Guardrails & Required File Templates
The build server rejects any track whose `Dockerfile` violates these rules:
- **MUST** use base images from `https://speeddemon.dev/allowlist.json` pinned by exact `@sha256:...` digest.
- **FORBIDDEN:** `# syntax=` lines, `ADD` with URLs/git repos, `COPY --from` referencing external images (`COPY --from=build` within a multi-stage Dockerfile is allowed), `--mount=` (cache/secret/ssh), `RUN --network`, `RUN --security`, `RUN --device`, `ONBUILD`, builds > 10 minutes, or final images > 2 GB.

### Required `track-<x>/Dockerfile` (Rust Multi-Stage Template)
```dockerfile
FROM rust:1-slim-trixie@sha256:1a939748a7cbab172de91a014e3804a97af1394a8c844faa67fce9401c4e40fb AS build
WORKDIR /src
COPY . .
RUN RUSTFLAGS="-C target-cpu=x86-64-v3" cargo build --release --offline

FROM debian:13-slim@sha256:62bd423aeee912639ca8ee681c2b762cb3e25137e0a92b20be455898a9f034ad
COPY --from=build /src/target/release/solution /app/solution
ENTRYPOINT ["/app/solution"]
```

### Required `track-<x>/Cargo.toml`
```toml
[package]
name = "solution"
version = "0.1.0"
edition = "2024"

[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
panic = "abort"
strip = true
```

### Required `track-<x>/.speed-demon.toml`
```toml
track = "a" # Set to "a", "b", "c", or "d"
team = "your-team-name"
language = "rust"
dependencies = 0
```

---

## 6. Track Specifications & Algorithmic Blueprints
Clone the official specs, generators, and naive references outside the submission repo so they do not bloat `scc` counts:
`git clone https://github.com/hackathon-raptors/speed-demon-2026 ../speed-demon-2026`

### Track A: Ingest (`track-a`) — The Parsing Track
- **Files:** Spec: `../speed-demon-2026/specs/a-ingest.md` | Ref: `../speed-demon-2026/reference/a_ingest.py` | Gen: `generators/a_ingest.py`
- **Dataset & Caps:** 239 MB, 3.1M space-delimited log lines. `T_ref = 32.05s` | Board Cap (3x): `96.15s` | Final Cap (10x): `321s`.
- **Output:** `/out/summary.txt`
- **Problem:** Aggregate structured log lines (RFC3339 microsecond timestamp, 7-value HTTP method enum, 1,842 distinct endpoints, status code 100-599, `uint32` microsecond latency) into per-endpoint request counts, p50/p95/p99 latency, and error rates bucketed by hour.
- **Optimization Strategy:**
  - `mmap` the input file and split into 12 newline-aligned chunks across threads (`std::thread::scope`).
  - Parse timestamps by fixed byte offsets (`YYYY-MM-DDTHH` is bytes `0..13`) and parse integer status/latency via branchless ASCII digit math.
  - Use fixed-size open-addressed hash tables (4,096 slots) per thread for the 1,842 endpoints.
  - Compute p50/p95/p99 using linear-time quickselect (`select_nth_unstable`) matching the exact index rounding of `a_ingest.py`.

### Track B: Index (`track-b`) — The Trade-Off Track
- **Files:** Spec: `../speed-demon-2026/specs/b-index.md` | Ref: `../speed-demon-2026/reference/b_index.py` | Gen: `generators/b_index.py`
- **Dataset & Caps:** 256 MB text corpus, 50,000 queries (62% `BOOL` with `AND`/`OR`/`NOT`, 38% `PHRASE` requiring exact token positions). `T_ref = 56.35s` | Board Cap (3x): `169.05s` | Final Cap (10x): `564s`.
- **Output:** `/out/results.txt`
- **Problem:** Build and query an inverted index timed together as a single wall time.
- **Optimization Strategy:**
  - **Query-Guided Indexing:** Because all 50,000 queries are available up-front, parse the query file FIRST to collect the unique query vocabulary.
  - While scanning the 256 MB corpus across 12 threads, completely ignore tokens not present in the queries.
  - Store token position lists ONLY for terms used in `PHRASE` queries; store compact document bitsets or sorted doc-ID arrays for `BOOL`-only terms. This slashes Peak RAM (`M`) and indexing time (`T`).

### Track C: Join (`track-c`) — The Correctness Track
- **Files:** Spec: `../speed-demon-2026/specs/c-join.md` | Ref: `../speed-demon-2026/reference/c_join.py` | Gen: `generators/c_join.py`
- **Dataset & Caps:** 204 MB hostile CSV, 3.32M records across 5 columns (`id`, `name`, `note`, `region_id`, `amount`). `T_ref = 14.44s` | Board Cap (3x): `43.32s` | Final Cap (10x): `144s`.
- **Output:** `/out/aggregates.txt`
- **Problem:** Parse a malformed/hostile CSV containing quoted commas, escaped quotes (`""`), embedded newlines inside quotes, mixed `\r\n` and `\n` line endings, and mid-file UTF-8 BOMs (`\xEF\xBB\xBF`), join against a lookup table, and emit sorted aggregates.
- **Optimization Strategy:**
  - Load the lookup table into a flat direct-indexed array or small open-addressed table.
  - Do NOT naively split on `\n`. Use a single-pass state-machine byte scanner (or quote-parity aware chunking) that strips mid-file `\xEF\xBB\xBF` BOMs on the fly and extracts only `region_id` and `amount`.
  - Match the exact numeric accumulation and sorting tie-breakers of `c_join.py`.

### Track D: Traverse (`track-d`) — The Algorithms Track
- **Files:** Spec: `../speed-demon-2026/specs/d-traverse.md` | Ref: `../speed-demon-2026/reference/d_traverse.py` | Gen: `generators/d_traverse.py`
- **Dataset & Caps:** 5M-edge directed weighted graph, 2,000 source-target shortest-path queries at bounded search distance. `T_ref = 72.69s` | Board Cap (3x): `218.07s` | Final Cap (10x): `727s`.
- **Output:** `/out/distances.txt`
- **Problem:** Parse the graph and answer all 2,000 shortest-path queries (preprocessing counts toward timed execution).
- **Optimization Strategy:**
  - Parse edges in parallel and build a flat **Compressed Sparse Row (CSR)** graph (`offsets: Vec<u32>`, `edges: Vec<(u32, u32)>`) for cache locality and minimal memory overhead.
  - Dispatch the 2,000 queries across 12 worker threads via an `AtomicUsize` work-stealing counter.
  - Give each thread a pre-allocated `dist` array and an epoch counter array (`visited_epoch: Vec<u32>`) incremented per query so clearing state between Dijkstra/bidirectional searches is `O(1)` instead of `O(V)`.

---

## 7. Local Testing, Verification & Logging Protocol
Follow this exact loop for every change:

1. **Read the Spec & Reference First:**
   Before coding any track, inspect `../speed-demon-2026/CONVENTIONS.md`, `specs/<track>.md`, and `reference/<track>.py`. Verify exact float formatting, sorting order, and edge cases.
2. **Generate Test Data Outside the Repo:**
   ```bash
   python3 ../speed-demon-2026/generators/a_ingest.py --seed 1 --scale sample --out ../data/a-sample
   python3 ../speed-demon-2026/generators/a_ingest.py --seed 1 --scale full --out ../data/a-full
   ```
3. **Run the Sandbox Verification Script (`check.py`):**
   ```bash
   python3 check.py a --data ../data/a-sample
   python3 check.py a --data ../data/a-full --expected <sha256>
   ```
   Also test against `--seed 2` through `--seed 5` to prevent overfitting before the holdout evaluation.
4. **Check Source Size (`S`) & Dependencies (`D`):**
   Run `scc track-<x>` and compare against `scc ../speed-demon-2026/reference/<track>.py`. Refactor repetitive code so `S <= S_ref` while keeping code clean and readable for human judges (27% weight).
5. **Update `PERF.md` (8% of Total Score) & `README.md`:**
   After every optimization and board push, log the hypothesis, architectural change, board run ID, wall time (`T`), peak memory (`M`), source LOC (`S`), and resulting `K` in `track-<x>/PERF.md`.
