# Git Forensic Dissection — oven-sh/bun PR-30412

**Operator:** Silas Locke
**Date of analysis:** 2026-05-14
**Target:** `/tmp/bun-repo`, branch `pr-30412` (blobless clone of `oven-sh/bun`)
**Claim under investigation:** Claude Code autonomously wrote/committed 1,009,257 lines of Rust in 10 days; 6,755 commits, many at the same Unix second.
**Method:** Raw object inspection via `git cat-file -p`, `diff-tree --numstat`, parent-chain walk. No trust in `git log` formatting — every claim is backed by the object store.

---

## BATCH A — 15-commit perf burst @ 1778548419 (2026-05-12T01:13:39Z)

### A.1 Raw object inspection

`git -C /tmp/bun-repo cat-file -p <sha>` on all 15. Full output captured. Summary table:

| # | SHA | tree | parent | author ts | committer ts | A==C | same person |
|---|-----|------|--------|-----------|--------------|------|-------------|
| 0 | 163ca8e5bf70 | 8a37b9ca5961 | 2412c07c469e | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 1 | 558296af88a1 | 4b4528964d7d | 163ca8e5bf70 | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 2 | 9ffe2bb2684e | b6acacb23c08 | 558296af88a1 | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 3 | 9b7eb9a7ebc4 | ebf3ab8af723 | 9ffe2bb2684e | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 4 | bf96f25dcea9 | b42a08117307 | 9b7eb9a7ebc4 | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 5 | b959256275eb | 5e1d1b30ad56 | bf96f25dcea9 | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 6 | 9532c8403b78 | f9265950c55c | b959256275eb | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 7 | da1353a87023 | 29f42994a52b | 9532c8403b78 | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 8 | 9bffefe9ffbc | 8bb7b9b7f1ee | da1353a87023 | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 9 | bfe6056b1e8e | 6e606a169f8f | 9bffefe9ffbc | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 10 | 564af07ee9cc | 49f903964aff | bfe6056b1e8e | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 11 | 44f61eeef714 | a3a4524b273f | 564af07ee9cc | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 12 | a9ad1e4f5976 | 823551ae8f41 | 44f61eeef714 | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 13 | 1a7bd1efefb6 | 5c02f94476b1 | a9ad1e4f5976 | 1778548419 +0000 | 1778548419 +0000 | YES | YES |
| 14 | d535eb180949 | 2db13e13074f | 1a7bd1efefb6 | 1778548419 +0000 | 1778548419 +0000 | YES | YES |

Author/committer for every commit: `Jarred Sumner <jarred@jarredsumner.com>`. **Author ts == committer ts on all 15. Author tz == committer tz == +0000 on all 15.**

### A.2 Parent chain verification

Verified programmatically. `sha[0].parent == 2412c07c469e` (a merge commit outside the batch), then `sha[1].parent == sha[0]`, `sha[2].parent == sha[1]`, ... `sha[14].parent == sha[13]`.

**Result: BATCH A IS A FULLY LINEAR, UNBROKEN CHAIN.** No breaks, no merges inside the batch, no forks.

Parent of the batch (`2412c07c469e`) is itself a merge commit at ts `1778540301` (2026-05-11T22:58:21Z) — **8,118 seconds (~2h15m) before** the batch. The batch's first commit is *not* contemporaneous with its parent.

### A.3 Per-commit numstat

`git -C /tmp/bun-repo diff-tree --numstat -r <sha>`:

| SHA | files | insertions | deletions | net |
|-----|-------|------------|-----------|-----|
| 163ca8e5bf70 | 3 | 76 | 15 | +61 |
| 558296af88a1 | 1 | 21 | 19 | +2 |
| 9ffe2bb2684e | 1 | 39 | 34 | +5 |
| 9b7eb9a7ebc4 | 1 | 73 | 63 | +10 |
| bf96f25dcea9 | 2 | 524 | 86 | +438 |
| b959256275eb | 1 | 26 | 0 | +26 |
| 9532c8403b78 | 3 | 711 | 166 | +545 |
| da1353a87023 | 2 | 111 | 130 | -19 |
| 9bffefe9ffbc | 3 | 235 | 201 | +34 |
| bfe6056b1e8e | 1 | 19 | 14 | +5 |
| 564af07ee9cc | 1 | 52 | 40 | +12 |
| 44f61eeef714 | 1 | 1 | 1 | 0 |
| a9ad1e4f5976 | 1 | 33 | 2 | +31 |
| 1a7bd1efefb6 | 2 | 155 | 13 | +142 |
| d535eb180949 | 2 | 93 | 56 | +37 |
| **TOTAL** | **25** | **2,169** | **840** | **+1,329** |

### A.4 First & last commit file listing

**163ca8e5bf70** (first): `src/bun_core/fmt.rs` (+34/-2), `src/bun_core/lib.rs` (+21/-10), `src/bun_core/result.rs` (+23/-3). 3 files, +76/-15. All inside `src/bun_core/` — coherent single-subsystem change.

**d535eb180949** (last): `src/sourcemap/Chunk.rs` (+1/-1), `src/sourcemap/InternalSourceMap.rs` (+92/-55). 2 files, +93/-56. All inside `src/sourcemap/` — coherent single-subsystem change.

### A.5 Full diff of the smallest commit (44f61eeef714, 1 file / 1 insertion / 1 deletion)

```diff
diff --git a/src/parsers/json_lexer.rs b/src/parsers/json_lexer.rs
index dff6963b54..26efaffb46 100644
--- a/src/parsers/json_lexer.rs
+++ b/src/parsers/json_lexer.rs
@@ -361,7 +361,7 @@ where
     // ── stepping ─────────────────────────────────────────────────────────
-    #[inline]
+    #[inline(always)]
     fn next_codepoint(&mut self) -> CodePoint {
```

This is a real, plausible micro-optimization: promoting `#[inline]` to `#[inline(always)]` on a lexer hot-loop function. The commit message ("json_lexer hot-loop — install/resolve benches") matches the diff exactly. Note the box-drawing comment (`── stepping ──`) — that is pre-existing surrounding context, not added by this commit.

### A.6 Timing analysis — does the chain prove sequential creation?

**YES — cryptographically, and this is the strongest hard fact in the whole investigation.**

A commit object's SHA-1 is computed over its full content, *including the parent line*. `sha[1]` contains `parent 163ca8e5bf70...` in its bytes. Therefore `sha[1]` could not have been hashed into existence until `sha[0]`'s hash already existed. By induction across all 15, the commits were necessarily *created in order* 0→14. This is not a timestamp claim — timestamps are attacker-controlled metadata; this is a hash-preimage dependency and cannot be faked without breaking SHA-1.

What the chain proves: **strict creation ordering.** What it does NOT prove: *wall-clock spacing.* The chain is fully consistent with all 15 `git commit` invocations firing inside a single second (15 × ~5–50 ms each fits comfortably in 1000 ms), and equally consistent with them being spread across an hour — the recorded timestamp was simply pinned. The chain rules out *parallel* creation; it says nothing about *elapsed time*.

### A.7 Clock-manipulation check

The timestamp `1778548419` is **suspicious by construction**, and here is the evidence:

1. **All 15 share the exact same second.** A natural `git commit` loop driven by a human or even a fast script would normally tick across 2–4 seconds for 15 commits unless the dates were explicitly pinned. Getting 15-for-15 on one second is the signature of `GIT_AUTHOR_DATE`/`GIT_COMMITTER_DATE` being set to a fixed value.
2. **author tz == committer tz == +0000 on all 15.** Jarred Sumner's other commits in this repo are overwhelmingly `-0700`/`-0800` (Pacific) — see PR-wide tz distribution in Synthesis. A sudden, uniform `+0000` across a burst is consistent with a scripted/automated environment (CI container, agent harness) or explicit env-var pinning, not an interactive shell on a Pacific-time workstation.
3. **author ts == committer ts exactly, on all 15.** For freshly-created commits this is normal — but combined with (1) and (2) it confirms these were minted in one automated pass, not rebased/cherry-picked later (a rebase would preserve author ts but bump committer ts).

**Verdict on A.7:** I cannot *prove* `GIT_*_DATE` was used — git does not record "this date was set by env var." But the evidence pattern (15/15 identical second + uniform +0000 + A==C) is exactly what date-pinning produces and is *not* what an unscripted commit sequence produces. **High confidence the timestamps were pinned/automated. This is benign in itself** — pinning commit dates is standard practice for reproducible automation and squash/replay workflows. It is a marker of automation, not of fraud.

### A.8 Content analysis — plausible for one AI session?

The 15 commits hit 15 distinct subsystems: bun_core, wyhash, base64, command_tag, clap, mimalloc_sys, bun_alloc, sys, ast, ast(again), js_parser, json_lexer, semver, resolver, sourcemap. Each diff is internally coherent and self-contained — the json_lexer one-liner is a textbook hot-path tweak, the bun_alloc one adds a whole `stack_fallback.rs` module (+429).

Is this plausible as one AI session? **Yes, and the diversity actually argues *for* batch-from-plan, not against it.** A perf sweep that walks a pre-built list of "hot subsystems" and applies one optimization per subsystem, committing after each, is a completely ordinary agent workflow. The subsystem diversity does not indicate anything sinister — it indicates the work was *planned* (a list of perf targets existed) and then *executed sequentially with one commit per target*. The single-second timestamp is the pinning artifact from A.7, not evidence the *thinking* happened in one second.

**What I cannot prove from git objects:** whether the diffs were generated live during this session or staged in advance and committed in a replay pass. Both produce byte-identical object graphs.

---

## BATCH B — divergence sweep @ 1778633685 (2026-05-13T00:54:45Z)

> **Correction to the brief:** the prompt states timestamp `1778774085` / `2026-05-13T00:54:45Z`. Those two are inconsistent — `1778774085` decodes to `2026-05-14T15:54:45Z`. The *actual* object timestamp on all 11 SHAs is **`1778633685` = 2026-05-13T00:54:45Z**. The date string in the brief is correct; the Unix number in the brief is wrong. I proceed with the verified object value `1778633685`.

### B.1 Raw object inspection

| SHA | tree | parent | author ts | committer ts | A==C |
|-----|------|--------|-----------|--------------|------|
| 4a306a213b91 | fdd976950a66 | c0326a9c293c | 1778633685 +0000 | 1778633685 +0000 | YES |
| 2d8f40d14ec5 | 94b35a93a6ec | 4a306a213b91 | 1778633685 +0000 | 1778633685 +0000 | YES |
| 54d1c288cdf1 | e70aba82b856 | 2d8f40d14ec5 | 1778633685 +0000 | 1778633685 +0000 | YES |
| b83da818d0ae | 6ef9a28fd06f | 54d1c288cdf1 | 1778633685 +0000 | 1778633685 +0000 | YES |
| f0aff8d471a2 | 4de2a2f6b69b | b83da818d0ae | 1778633685 +0000 | 1778633685 +0000 | YES |
| b57f5f33f142 | 6e882db1d3b8 | f0aff8d471a2 | 1778633685 +0000 | 1778633685 +0000 | YES |
| ab95af0ddcec | e929f54cfa56 | b57f5f33f142 | 1778633685 +0000 | 1778633685 +0000 | YES |
| 7cd6203ed2c9 | 4a7a8d3bc527 | ab95af0ddcec | 1778633685 +0000 | 1778633685 +0000 | YES |
| fa2d7e655650 | d59405a58d57 | 7cd6203ed2c9 | 1778633685 +0000 | 1778633685 +0000 | YES |
| 5c9a35f7879c | 68d40e16da0d | fa2d7e655650 | 1778633685 +0000 | 1778633685 +0000 | YES |
| 7027892a16cb | 2ef5c4aa2774 | 5c9a35f7879c | 1778633685 +0000 | 1778633685 +0000 | YES |

All authored & committed by `Jarred Sumner <jarred@jarredsumner.com>`, tz `+0000`, A==C on all 11. Parent chain: **FULLY LINEAR** — verified `sha[0].parent == c0326a9c293c`, then each subsequent parent == predecessor. All 11 tree hashes distinct.

**Sharp finding — the batch is bigger than the brief states.** The parent `c0326a9c293c` is itself `divergence(paths): fix 3 paths/resolver sites` at ts **`1778633684`** — exactly *one second earlier*. Walking back: `3470adf1e072 divergence(ast)` and `e5a75c2d0a0c divergence(core)` are also at `1778633684`. So the "divergence sweep" is at least 14 commits spanning a **2-second window** (`...684` and `...685`), not an isolated 11-commit/1-second event. The sweep crosses a second boundary mid-run — which is itself mild evidence *against* hard date-pinning for this batch and *for* a genuine fast loop (see B.5).

### B.2 Subsystem-label verification

`git show --stat` on 3 commits — labels vs. actual file paths:

- **2d8f40d14ec5** `divergence(http): fix 6 http client sites` → touches 6 files, *all* under `src/http/` (`HeaderBuilder.rs`, `Headers.rs`, `InternalState.rs`, `h3_client/AltSvc.rs`, `h3_client/ClientContext.rs`, `lib.rs`). **6 files, label says "6 sites". Match.**
- **f0aff8d471a2** `divergence(jsc): fix 5 jsc sites` → 5 files, all under `src/jsc/`. **Match.**
- **7027892a16cb** `divergence(socket): fix 2 runtime/socket sites` → 2 files, both under `src/runtime/socket/`. **Match.**

Cross-checked the numstat for all 11: the file-count per commit corresponds to its label number in most cases (sys=3 files, http=6, install=10, bundler=5, jsc=5, server=3, api=4, bake=2, cli=2, shell=2, socket=2). The "N sites" in the message tracks "N files touched" closely. **Labels are accurate, not decorative.**

### B.3 What does the tree-hash sequence tell us about Option A/B/C?

All 11 trees are distinct and each commit's tree is reachable only by applying that commit's diff to the parent's tree. That confirms **each commit is a real, distinct snapshot** — not 11 pointers to one tree, not empty commits.

But here is the limit of what git can tell you: **the object graph for Option A (a `for subsystem in ...; do fix; git commit; done` loop) and Option B (11 files prepared in advance, then `git add x && git commit` in rapid succession) is byte-for-byte identical.** Git records the *result* of each commit, never the *process* that produced the working-tree state. The linear chain proves sequential commit creation; it cannot distinguish "fixed then committed" from "pre-staged then committed."

What slightly tips it: the sweep crossing the `...684`/`...685` second boundary (B.1) is more consistent with **Option A** — a real loop running at machine speed and naturally ticking the clock — than with a single pinned-date replay (which would land everything on one second, as Batch A did). Not conclusive. Leaning A.

### B.4 Full diff of 54d1c288cdf1 (claims "fix 10 install/lockfile sites")

Full diff captured. It touches **10 files** under `src/install/`: `NetworkTask.rs`, `PackageInstall.rs`, `PackageManagerTask.rs`, `dependency.rs`, `lockfile/Package.rs`, `lockfile/Package/Scripts.rs`, `lockfile/printer/Yarn.rs`, `resolution.rs`, `resolvers/folder_resolver.rs`, `yarn.rs`. So "10 sites" == 10 files.

**Is it 10 distinct fixes or a templated transformation?** It is a *thematic* sweep — one conceptual bug ("Zig `writeAll` writes raw bytes; the Rust `Display`/`BStr` path is lossy on non-UTF-8") applied to its ~10 manifestation sites. But it is **not** a mechanical find-replace:

- `NetworkTask.rs`: replaces `append_fmt(... format_args!("Bearer {}", BStr::new(...)))` with `append_bytes_value("Authorization", b"Bearer ", &scope.token)` — a real API change.
- `PackageInstall.rs`: **reorders local-variable declarations** so a `scopeguard::defer!` drops in the correct order relative to a `Box` reclaim — a Rust drop-order semantics fix, completely different in kind from the byte-encoding fix.
- `resolution.rs`: adds two whole new `write_to<W>` methods (+134 lines) implementing byte-exact serialization.
- `Package.rs`: threads a new `&bump: &bun_alloc::Arena` lifetime parameter through ~30 call sites of `as_utf8` — a signature change with borrow-checker implications.

Each carries a hand-reasoned `// PORT NOTE:` explaining the Zig-vs-Rust semantic gap. **This is the work of an agent (or human) that understands Zig drop semantics, Rust borrow lifetimes, and UTF-8 vs. raw-byte I/O — not a templated transform.** Whoever produced it was reasoning about correctness, not pattern-substituting.

### B.5 Timing arithmetic

11 commits in 1 second (or 14 in 2 seconds for the full sweep). At ~5–50 ms per `git commit` invocation, 11 commits = 55–550 ms of git overhead. That fits in one second with room to spare.

- **Humanly type-able?** No. Not a chance. No human types `git commit` 11 times — let alone edits 11 subsystems — inside one second.
- **Automation-possible?** Trivially. A shell loop or an agent harness calling git in sequence does this comfortably. The *commit* operations are fast; what cannot happen in 1 second is the *editing*. Therefore the edits were necessarily completed *before* the commit burst began — whether by a loop that edited-then-committed each subsystem fast (Option A, with editing time absorbed earlier in the sweep) or by pre-staging (Option B).
- **The limit:** git itself has no rate limit here. The binding constraint is human/AI *thinking and editing* time, and git objects cannot measure that — they only capture the commit instants.

---

## BATCH C — Engineering plan (phase-b0 → phase-b1)

### C.1 Raw object inspection

| SHA | tree | parent | author ts | committer ts | A==C | UTC |
|-----|------|--------|-----------|--------------|------|-----|
| 2050a922ff00 | 56511e17b4af | ffa6ce211a02 | 1777939227 | 1777939227 | YES | 2026-05-05T00:00:27Z |
| e024d38c7a71 | cb282f0f5aed | 4723dc28db58 | 1777941115 | 1777941115 | YES | 2026-05-05T00:31:55Z |
| a7fb8f73dd34 | d7a5d8f0fcb4 | 372275f7581a | 1777942819 | 1777942819 | YES | 2026-05-05T01:00:19Z |
| 4244fa859e80 | 48e9c1d5de9d | a7fb8f73dd34 | 1777943546 | 1777943546 | YES | 2026-05-05T01:12:26Z |
| cad2ea67dc30 | e1bc4fd31a92 | 4244fa859e80 | 1777943830 | 1777943830 | YES | 2026-05-05T01:17:10Z |
| f5d5bc97939d | 2be3abfd3326 | cad2ea67dc30 | 1777944589 | 1777944589 | YES | 2026-05-05T01:29:49Z |
| 3157cb14b597 | da35bad1081b | f5d5bc97939d | 1777944987 | 1777944987 | YES | 2026-05-05T01:36:27Z |
| de79331ef912 | a36300b84c59 | 3157cb14b597 | 1777954151 | 1777954151 | YES | 2026-05-05T04:09:11Z |
| 24883e4038f7 | af4c84d02da4 | de79331ef912 | 1777954221 | 1777954221 | YES | 2026-05-05T04:10:21Z |
| 61066a9e1fbb | fecb2fc250a3 | 24883e4038f7 | 1777954452 | 1777954452 | YES | 2026-05-05T04:14:12Z |

All `Jarred Sumner <jarred@jarredsumner.com>`, tz `+0000`. **A==C on all 10** — consistent with fresh commits, not rebased.

### C.2 Chain analysis — and a finding the brief did not anticipate

**The 10 listed commits do NOT form a contiguous chain.** Two parent links point *outside* the list:

- `e024d38c7a71.parent == 4723dc28db58` — **not** `2050a922ff00`.
- `a7fb8f73dd34.parent == 372275f7581a` — **not** `e024d38c7a71`.

Walking the real chain fills the gaps with interleaved `docs:` commits:

```
2050a922ff00  00:00:27  phase-b0: add crate DAG analyzer
4723dc28db58  00:30:07  docs: add Dispatch section (vtable/hoisted-match for cross-tier union(enum))
e024d38c7a71  00:31:55  phase-b0: remove spurious back-edge imports (DELETE pass, 99 edits)
2665eea1d5f3  00:35:49  docs: add CYCLEBREAK.md (B-0 per-crate move-out/move-in spec)
372275f7581a  00:42:57  docs: add Concurrency section; remap CYCLEBREAK new-leaf targets
a7fb8f73dd34  01:00:19  phase-b0: move-out pass — 36 verified-clean crates
4244fa859e80  01:12:26  phase-b0: move-out pass — 4 fixed crates
cad2ea67dc30  01:17:10  phase-b0: move-out pass — final 4 crates
f5d5bc97939d  01:29:49  phase-b0: move-in pass — 33 target crates (579 symbols)
3157cb14b597  01:36:27  phase-b0: collapse tier-6 into runtime/
de79331ef912  04:09:11  phase-b1: scaffold Cargo workspace (96 crates)
24883e4038f7  04:10:21  phase-b1: prune back-edge deps from Cargo.toml
61066a9e1fbb  04:14:12  phase-b1: bun_alloc compiles
```

**This is exculpatory, not incriminating.** The "31-minute gap" the brief flagged (`00:00:27` → `00:31:55`) is *not* a gap — it is filled by `4723dc28db58 docs: add Dispatch section` at `00:30:07`. The agent spent that half hour writing a design document (`docs:` commit) before doing the DELETE pass. The whole-chain timeline is fully contiguous and densely populated. The "gaps" were an artifact of looking only at the `phase-b0`-prefixed subset.

### C.3 The "crate DAG analyzer" — what is it?

`git show --stat 2050a922ff00`: **one file** — `scripts/crate-dag.ts`, +175 lines, 0 deletions.

`git show 2050a922ff00:scripts/crate-dag.ts` — it is a real, executable Bun/TypeScript program:

```ts
#!/usr/bin/env bun
// Compute crate DAG, intended tiers, and back-edges for Phase B-0.
import { readdirSync, readFileSync, statSync, writeFileSync } from "node:fs";
...
const dirs = readdirSync("src").filter(d => { ... isDirectory() ... });
// crate-ref → src dir (PORTING.md §Crate map; default bun_X → X)
const special: Record<string, string> = { str: "string", output: "bun_core", ... };
// Intended tier (from restructure plan). Anything *_sys = 0. *_jsc = 6.
const TIER: Record<string, number> = { bun_core: 0, bun_alloc: 0, wyhash: 0, ... };
```

It scans `src/`, maps crate references to directories, assigns each crate a dependency tier (T0 zero-dep primitives → T6 subsystems), and computes the dependency DAG and its back-edges. **This is exactly the tool you write *first* when you are about to restructure a monolith into an acyclic Cargo workspace.** You need to know the tier ordering and which imports create cycles before you can move anything. The agent built its own instrumentation before touching the codebase. That is a hallmark of competent migration work.

### C.4 Do the claimed numbers match the diffs?

`git diff-tree --numstat -r`:

- **e024d38c7a71** "DELETE pass, 99 edits": touches **67 files**, +103/-110 = 213 changed lines. The message says "99 edits" — that does not map to file count (67) or to insertions/deletions directly. "99 edits" most plausibly = 99 discrete hunks/import-line removals across the 67 files. The numbers are in the right ballpark (213 changed lines across 67 files comfortably contains ~99 distinct edit sites) but I **cannot verify the exact "99" from numstat alone** — it would require hunk-counting. Not contradicted; not precisely confirmed.
- **a7fb8f73dd34** "36 verified-clean crates": touches **115 files**, +1916/-1383, across **36 distinct top-level `src/` subdirectories** (aio, analytics, boringssl, bun_alloc, crash_handler, csrf, css, dotenv, errno, event_loop, glob, http, http_types, ini, interchange, js_parser, js_printer, libarchive_sys, options_types, paths, picohttp, ptr, resolver, router, s3_signing, semver, sha_hmac, shell_parser, sourcemap, string, sys, threading, url, uws_sys, watcher, windows_sys). **Counted: 36 directories. The "36 crates" claim is exactly confirmed by the file diff.**

### C.5 de79331ef912 — "scaffold Cargo workspace (96 crates)"

`git show --stat`: **151 files, +3184/-0** (pure addition, nothing deleted). Of those:
- **97 `Cargo.toml` files** added (1 root workspace manifest + 96 per-crate manifests). **"96 crates" claim confirmed** — 96 member crates + the workspace root = 97 manifests.
- **52 `lib.rs`** files touched (autogen'd `mod` declarations, as the message says).

Content sample — root `Cargo.toml`:
```toml
[workspace]
resolver = "2"
members = [ "src/analytics", "src/api", "src/base64", "src/bundler", ... ]
```
Per-crate `src/bun_alloc/Cargo.toml`:
```toml
[package]
name = "bun_alloc"
version.workspace = true
edition.workspace = true
[lib]
path = "lib.rs"
[dependencies]
parking_lot.workspace = true
... bun_core.workspace = true
bun_collections.workspace = true
bun_ptr.workspace = true
...
```
These are **real, well-formed Cargo manifests** with proper `version.workspace = true` inheritance, `[lib] path` overrides, and crate-internal `bun_*` deps wired through `.workspace = true`. Not stubs, not placeholders. `src/aio/Cargo.toml` similarly declares `name = "bun_aio"` with a coherent dependency set (bun_core, bun_io, bun_string, bun_sys, bun_threading, bun_uws_sys).

### C.6 Engineering-plan assessment

The phase-b0/b1 sequence, in order:

1. **Write a DAG analyzer** (`crate-dag.ts`) — instrument before you touch anything.
2. **Write design docs** (`docs: Dispatch`, `docs: CYCLEBREAK.md`, `docs: Concurrency`) — spec the cross-crate dispatch/cycle-breaking strategy.
3. **DELETE pass** — remove spurious back-edge imports that create cycles (99 edits).
4. **Move-out passes** — relocate 36 + 4 + 4 = 44 verified-clean crates to their tiers.
5. **Move-in pass** — pull 579 symbols into 33 target crates.
6. **Collapse tier-6** — fold bake/shell/test_runner/cli/napi into `runtime/`.
7. **Scaffold the Cargo workspace** — 96 crate manifests + autogen'd mod decls.
8. **Prune back-edge deps** from the generated `Cargo.toml`s to make the workspace acyclic.
9. **Compile each crate, lowest tier first** — bun_alloc → bun_core → windows_sys → collections → unicode → base64/platform → errno → ... each commit is one "compiles" milestone.

**This is a textbook, expert-grade monolith-to-workspace migration.** The specific moves — building a tier model, breaking dependency cycles *before* splitting, bottom-up tier-ordered compilation — are exactly what an experienced systems engineer doing a large Rust workspace decomposition would do. It is not improvised; it follows the standard "topological-sort-then-extract" playbook. Whoever (or whatever) designed this either has done large-scale Rust migrations before or was working from a well-understood methodology.

### C.7 phase-b1 timing — real compile/fix loop?

The brief's claimed window was approximate. The actual `phase-b1` sequence (grepped from the object store):

```
04:09:11  scaffold Cargo workspace (96 crates)
04:10:21  prune back-edge deps          (+70s)
04:14:12  bun_alloc compiles            (+231s)
04:17:40  bun_core compiles             (+208s)
04:18:14  windows_sys compiles          (+34s)
04:19:35  collections compiles          (+81s)
04:20:37  unicode compiles              (+62s)
04:22:26  base64+platform compile       (+109s)
04:25:30  errno compiles                (+184s)
04:26:39  uws_sys + 5 *_sys compile     (+69s)
04:27:48  ptr + safety compile          (+69s)
04:30:09  paths + string compile        (+141s)
04:31:01  sys compiles                  (+52s)
   ... then a gap to 06:27:41 for tier-2 batch ...
```

Inter-commit gaps run 34 s to 231 s — **irregular, content-correlated spacing.** `windows_sys compiles` took 34 s (small crate); `bun_alloc compiles` took 231 s (the foundational allocator, hardest). A pinned-date replay would produce uniform or single-second spacing. **This irregular, plausibly-proportional cadence is exactly what a genuine `compile → read errors → fix → recompile` loop looks like.** Of the three batches, phase-b1 is the one whose timestamps I read as *organic* rather than pinned — the variance is the tell.

Note also: each commit message names the *specific* crate and the *specific* gating done ("gate BSS/heap_breakdown/scope; add Alignment/AllocatorVTable") — the kind of detail that only exists if a real compiler was producing real errors that were really being fixed.

---

## SYNTHESIS

### Anomaly table

| # | Finding | Batch | Severity | Explanation |
|---|---------|-------|----------|-------------|
| 1 | 15 commits share exact Unix second `1778548419`; tz uniformly `+0000` | A | MEDIUM | Signature of `GIT_*_DATE` pinning or automated harness. Benign per se, but proves the timestamps are not organic wall-clock. |
| 2 | Author == committer timestamp on 100% of all 36 batch commits | A,B,C | LOW | Normal for fresh commits; combined with #1 confirms no post-hoc rebase. Not suspicious alone. |
| 3 | Brief's stated Unix ts for Batch B (`1778774085`) does not match the object store (`1778633685`) | B | LOW (brief error) | The date *string* in the brief was right; the *number* was wrong. Real objects verified at `2026-05-13T00:54:45Z`. |
| 4 | "Divergence sweep" is larger than 11 commits — extends to `1778633684`, crossing a second boundary | B | LOW | More commits at the prior second (divergence core/ast/paths). Sweep spans ≥2 s, ≥14 commits. The boundary-crossing weakly argues *against* hard pinning for batch B. |
| 5 | Batch C's 10 listed SHAs are NOT a contiguous chain — 2 parent links point to interleaved `docs:` commits | C | LOW (exculpatory) | The brief's "31-minute gap" is filled by `docs: add Dispatch section`. Real chain is contiguous and dense. |
| 6 | `crate-dag.ts` — agent wrote its own 175-line tiering/cycle-detection tool before migrating | C | INFORMATIONAL | Strong positive signal of competent, instrumented engineering — not an anomaly of fraud, an anomaly of *quality*. |
| 7 | Batch B install diff carries hand-reasoned `// PORT NOTE:` comments on Zig-vs-Rust drop order, borrow lifetimes, UTF-8 semantics | B | INFORMATIONAL | Content reflects genuine semantic reasoning, not templated substitution. Argues for real authorship (human or capable agent), against mechanical generation. |
| 8 | phase-b1 inter-commit gaps are irregular (34 s–231 s) and roughly proportional to crate complexity | C | INFORMATIONAL | Consistent with a real compile/fix loop. The *one* batch whose timestamps read as organic. |
| 9 | All 6,651 `+0000` Jarred-authored commits diverge from his historical `-0700/-0800` Pacific tz | PR-wide | MEDIUM | The entire PR's working timezone is `+0000`, unlike Jarred's pre-2026 history. Consistent with an automated/CI/agent environment. Not fraud — a fingerprint of the harness. |

### What CAN be proven from git objects alone

- **Strict creation order** of every commit in Batches A and B (SHA-1 parent-preimage dependency — uncircumventable).
- Batches A and B are **fully linear, unbroken chains**; Batch C is linear once the interleaved `docs:` commits are included.
- Every commit is a **real, distinct tree snapshot** — no empty commits, no duplicate-tree pointers.
- The **claimed file/crate counts are accurate** where countable: "36 crates" = 36 dirs (exact), "96 crates" = 96 manifests (exact), "10 install sites" = 10 files (exact), subsystem labels match file paths (exact).
- The **diffs are coherent and semantically reasoned**, not templated transforms.
- Author == committer timestamps everywhere → **no post-hoc rebasing** of these batches.

### What CANNOT be proven from git objects alone

1. **Whether the code was AI-generated or human-written.** The object store records *results*, never *authorship process*. A capable human with a script and a capable agent produce identical object graphs.
2. **Whether diffs were generated live in-session or pre-staged and replayed.** Option A (edit-then-commit loop) and Option B (pre-stage, then batch-commit) are byte-identical in git.
3. **Actual wall-clock elapsed time.** Timestamps are attacker-controlled metadata. The single-second batches *could* have taken one second or one week of real work — the chain only fixes ordering, not duration.
4. **Whether `GIT_AUTHOR_DATE`/`GIT_COMMITTER_DATE` were explicitly set.** Git does not record the provenance of a date. We infer pinning from the *pattern* (15/15 identical second), but cannot read it from an object field.
5. **Who designed the migration plan.** The plan in Batch C is expert-grade, but git cannot tell us if it came from an AI, a human architect, a prior internal design doc, or all three.

### Verdict on the "parallel commit" skepticism

**Clock manipulation — can I rule it in or out?**
I can rule *in* **timestamp pinning/normalization** for Batch A with high confidence (15/15 commits on one second + uniform `+0000` is not what unscripted commits produce). I **cannot** rule in *malicious* clock manipulation — pinning commit dates is a standard, legitimate practice for automated/reproducible workflows, and there is no evidence the dates were moved to *deceive* (e.g., no commit with a committer-date *earlier* than its parent, which would be the smoking gun of fraud — I checked: every batch's parent timestamp is ≤ the batch timestamp). **Conclusion: timestamps were automated/pinned (benign), not fraudulently manipulated.**

**Parallel commits — ruled OUT.** The SHA-1 parent-preimage chain makes parallel commit creation cryptographically impossible. The commits in A and B were created strictly sequentially. The "many commits at the same second" is not parallelism — it is fast *sequential* commits with pinned (Batch A) or fast-ticking (Batch B) dates. The skepticism about "parallel commits" rests on a misunderstanding of what a shared timestamp means: it means *fast or pinned*, never *parallel*.

**Pre-generation + batch commit — can I rule it in or out?**
**I cannot rule it out, and I cannot rule it in.** This is the genuine limit of git forensics. Batch A's single-second pinning is *consistent* with pre-generation + replay, but also consistent with a fast edit-commit loop with pinned dates. Batch B's second-boundary crossing leans toward a live loop. Batch C's irregular phase-b1 cadence leans hard toward a live compile/fix loop. **My read across all three: more evidence of live automated loops than of static pre-generation — but git objects cannot make this definitive.**

**What definitive proof would look like:** It is not in the git objects. It would require *out-of-band* evidence — the agent's session transcript / tool-call log, CI build logs with their own independent timestamps, the reflog of the machine that created the commits, filesystem mtimes on the working tree during the session, or the harness's own execution records. Git is a content-addressed *result* store; it was never designed to prove *process*. Anyone expecting git objects alone to settle "did an AI write this" is asking the wrong tool.

### Engineering plan assessment

The phase-b0/b1 sequence reveals a **coherent, expert-level software migration strategy** — and this is the single most informative finding in the investigation. The sequence is:

> build instrumentation (DAG analyzer) → write design specs (Dispatch/CYCLEBREAK/Concurrency docs) → break dependency cycles (DELETE pass) → relocate verified-clean crates tier by tier (move-out) → consolidate symbols (move-in) → collapse the top tier → scaffold the Cargo workspace → prune back-edges to enforce acyclicity → compile bottom-up, one crate per commit, fixing as you go.

That is the canonical "topological-sort-then-extract" playbook for decomposing a monolith into an acyclic multi-crate workspace. It is *not* improvised — it follows a known methodology with discipline: instrument first, spec second, break cycles *before* splitting, compile in dependency order. The `crate-dag.ts` tool and the three `docs:` commits prove the planning was *explicit and externalized*, not implicit.

**What that implies about who or what designed it:** Whoever or whatever drove this either (a) has prior experience with large-scale Rust workspace migrations, or (b) was executing from a well-defined methodology / internal design document. An LLM agent operating from a good plan is fully capable of executing this — the work is mechanically demanding but methodologically standard. The plan's *quality* is real and verifiable from the object store; the plan's *origin* (autonomous AI reasoning vs. human-authored methodology the AI followed) is exactly the thing git cannot tell us. The migration is competent. Whether the competence is the model's or a human's flowing through the model is the open question — and it stays open after this investigation.

---

## Operator's bottom line

Nothing in the git object store indicates fraud. Every checkable claim — file counts, crate counts, chain integrity, subsystem labels — **checks out**. The "suspicious" same-second batches are explained by timestamp pinning/automation, which is benign and is *not* parallelism (the SHA chains prove strict sequential creation). The engineering is real, coherent, and expert-grade.

What the git objects **cannot** establish is the one thing the headline claim hinges on: whether an AI *autonomously generated* this, or whether it executed a human-authored plan, or whether code was pre-staged and replayed. Those questions require the session transcript and the harness logs — not the repository. I have proven what is provable. The rest is out of scope for git forensics, and anyone who tells you the commit graph alone settles the "autonomous AI" question is overselling the evidence.
