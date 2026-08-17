# TeddyClowers

TeddyClowers is a privacy-first content-filtering engine implemented as a Rust workspace with a browser adapter boundary in TypeScript. The engine is intentionally headless: it parses and compiles filter lists, evaluates network requests, exposes diagnostics, and supports bounded in-memory caching without sending browsing data anywhere.

> **Current status:** the core parser, compiler, indexed matcher, exception handling, diagnostics, cache, starter CLI, unit tests, and browser adapter contract are **IMPLEMENTED and TESTED**. The project is not yet **PRODUCTION READY**: full filter-ecosystem compatibility, fuzzing, browser packaging, Android integration, and reproducible benchmark baselines remain future milestones.

## Workspace

| Area | Location | Status |
|---|---|---|
| Rust core parser/compiler/matcher/cache | `crates/teddyclowers-core` | Implemented, tested |
| Rust CLI | `crates/teddyclowers-cli` | Implemented, tested by workspace build |
| Chromium extension package | `browser/chromium` and `dist/teddyclowers-chromium-v0.2.0.zip` | Store-oriented MV3 candidate |
| Firefox extension package | `browser/firefox` and `dist/teddyclowers-firefox-v0.2.0.zip` | AMO-oriented MV3 candidate |
| Browser adapter boundary | `browser/chromium/src/index.ts` | Typed bridge contract |
| Starter rules | `rules/default.txt` | Example rules only |
| Architecture and security notes | `docs/` | Implemented documentation |
| Fuzzing, Android, full cosmetic DOM runtime | `fuzz/`, `android/`, runtime adapters | Planned |

## Build and test

Install Rust 1.75 or newer, then run:

```bash
cargo test
cargo build --release
```

The release profile enables thin link-time optimization, one code-generation unit, panic aborts, and stripped binaries. These are build settings, not performance claims; benchmark results must be collected on the target hardware and workload.

## Browser extension packages

Run `./package_extensions.sh` to generate the Chrome and Firefox archives. The packages contain a Manifest V3 service worker, compiled Declarative Net Request rules, popup controls, an options page, a bounded cosmetic-filter content script, local allowlist and user blocklist settings, deterministic store icons, and privacy/listing drafts under `store/`.

The archives are **submission candidates**, not platform approval guarantees. Replace publisher and support placeholders, host the privacy policy at a public HTTPS URL, review filter-list licensing, upload through the appropriate store dashboard, and resolve validator or reviewer feedback.

## CLI

The CLI uses a small dependency-free argument parser to keep startup and compile cost low.

```bash
cargo run -p teddyclowers-cli -- check https://doubleclick.net/ads.js --type=script --third-party
cargo run -p teddyclowers-cli -- explain https://www.google-analytics.com/collect --type=xmlhttprequest --rules=rules/default.txt
cargo run -p teddyclowers-cli -- inspect '||ads.example^$script,third-party,domain=example.com'
cargo run -p teddyclowers-cli -- test rules/default.txt
cargo run -p teddyclowers-cli -- compile rules/default.txt
cargo run -p teddyclowers-cli -- benchmark --rules=rules/default.txt --iterations=100000
cargo run -p teddyclowers-cli -- update --rules=rules/default.txt
```

The CLI deliberately does not fetch remote lists. Fetching, signature validation, persistence, scheduling, and policy decisions belong to an outer runtime that can be audited separately.

## Supported rule subset

The initial parser accepts host-anchored and URL substring rules, `@@` exceptions, domain restrictions, resource-type restrictions, third-party restrictions, `important`, safe named redirects, and basic cosmetic/scriptlet classification. Regex rules are rejected until the engine has bounded deterministic support. Unknown options are rejected rather than silently misinterpreted.

Cosmetic rules are classified and retained in the compiled snapshot. The packaged extension applies the supported selector subset through a bounded content script; the Rust core remains browser-independent.

## Design decisions

The hot path is staged: request normalization, bounded cache lookup, token-index candidate selection, cheap context checks, URL matching, exception evaluation, and final decision. Compiled rule snapshots are immutable and replaceable through `Engine::replace_rules`; replacing a snapshot clears the in-memory decision cache so stale decisions are not reused.

The engine has no network client, persistent history store, analytics, user account system, or default telemetry. Cache keys include URL and request context, and the cache is bounded and process-local. This is a privacy property, not a substitute for a browser's privacy model.

## Honest engineering status

No comparative performance number is claimed here. The CLI benchmark reports only the local execution environment's observed elapsed time, throughput, average decision time, and cache counters. A future benchmark report must include hardware, operating system, compiler, optimization flags, list sizes, workloads, iteration count, and statistical summary before making comparisons.

## License

The repository is structured for GNU GENERAL 3.0 licensing. 
©Ottahen
©PrimeAct
