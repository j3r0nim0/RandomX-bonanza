# VoltMiner fork of RandomX-bonanza

This fork exists to preserve a **working, shipped** WebAssembly RandomX engine.

Context: VoltMiner's browser miner (web.voltminer.net) originally ran its own
WebRandomX-based engine. To get the speed of a JIT it vendored
`aa022/RandomX-bonanza` instead, and that is what ships today in
`js/vendor/randomx/web-randomx-mt.{js,wasm}`. This branch is that engine, kept so
the working version cannot be lost, and so the next line of work — rebuilding the
JIT on top of our own engine — has something to diff against.

## Branches

| Branch | What it is |
|---|---|
| `voltminer-jit` | The shipped engine: upstream `wasm/` sources + our overlay. **This is the working version.** |
| `upstream-f9848e3` | The exact commit we built from, preserved here. |

### Why `upstream-f9848e3` exists

Upstream force-pushed a history rewrite ("deslop history") and **`f9848e3` is no
longer reachable** from `aa022/RandomX-bonanza`. Our published transparency pages
cite that SHA, so a verifier following them would land on a clone with no such
commit. It is preserved here as a branch.

The rewrite did **not** touch the build inputs: `git diff` of the whole `wasm/`
subtree between `f9848e3` and upstream's current HEAD is empty — every blob is
byte-identical. What changed is bonanza's JS/site scaffolding (`dist/`, `bench/`,
`tests/`, `public/`, `proxy/`, `Makefile`, `README.md`). So the build still
reproduces from upstream HEAD; only the citation is dangling.

## What we add on top of upstream

Three files, ~600 lines — an adapter, not engine work. The wasm JIT itself is
upstream's, unmodified:

| Path | What it is |
|---|---|
| `wasm/src/src/web_randomx.cpp` | API shim: little-endian nonce at blob byte 39, a pthread `_randomx_mine` path over one shared dataset, the chunked dataset API (begin/step/ptr/items) JS drives the fill with, and the `_randomx_*` exports |
| `wasm/src/src/web_randomx.h` | Its header |
| `wasm/build.sh` | +25/−7 lines: shim added to `SOURCES`, our export names, pthread link retargeted to `web-randomx-mt.js` |

`git diff` of this branch against upstream touches **no `wasm_jit_*` file** — the
JIT backend is upstream's work, byte-for-byte.

`voltminer-overlay/` holds the same three files as a standalone patch plus the
overlay README, for patching a fresh clone by hand.

## Build

    ./build-web-randomx-mt.sh

Docker + Emscripten 4.0.10. Memory stays **growable** — upstream's own default
(`ALLOW_MEMORY_GROWTH=1`, `MAXIMUM_MEMORY=4 GiB`). Do not reintroduce a fixed
heap: an earlier revision of ours forced `FIXED_MEMORY=1` at 3584 MiB, which
committed 3.5 GB on load in *every* mode and swapped a 16 GB machine to death.
Growable commits 16 MiB in light mode and grows to about 2.4 GB for a
full-dataset run.

## Licences

- **wasm JIT backend** (`wasm_jit_*.{c,h}`, `wasm_jit_fuse_table.h`,
  `rx_aes_relaxed_blob*.h`, and the RandomX core edits) — **CC0 1.0**, dedicated
  by aa022. No obligations.
- **RandomX engine** — BSD-3-Clause, © tevador and The Monero Project.
- **`src/jit_stubs/semifloat.c` and `mulh.c`** — **not** aa022's and **not**
  covered by the CC0. Byte-identical to `l1mey112/randomx.js`, BSD-3-Clause,
  © tevador `<tevador@gmail.com>` and l-m `<l-m@l-m.dev>`. Their notice must be
  retained. This is the one attribution obligation in the tree.
