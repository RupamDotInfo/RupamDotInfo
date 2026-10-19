# Rupam Bhaduri

**Software Engineer · Product Builder · Rust, Laravel, Vue/TypeScript**

I build software end to end: application architecture, backend services, desktop interfaces, testing and delivery. I care about systems that behave correctly under failure, tests that catch real defects, and releases whose claims can be independently checked.

My current engineering interests are **systems software, backend/product engineering, cross-platform desktop applications, and software reliability**.

## Current projects

| Project | Focus | Status |
|---|---|---|
| [**prerender-audit**](https://github.com/RupamDotInfo/prerender-audit) | Zero-dependency Node.js CLI that checks whether sitemap URLs and static prerendered HTML agree | Public source; released as [v1.0.0](https://github.com/RupamDotInfo/prerender-audit/releases/tag/v1.0.0) on GitHub. Not published to npm. |
| **LanFy** | Rust-based LAN messenger focused on secure local communication and peer discovery | In development; source not public yet |
| **DownFy** | Cross-platform download manager with a Rust engine and native mobile work | In development; source not public yet |
| **TorrentFy** | BitTorrent desktop client built around a Rust engine | In development; not release-ready |
| **OfficeFy** | Desktop productivity suite; current work emphasizes document fidelity and pagination | In development |
| **AcadeFy** | Education software and multi-tenant SaaS | Private product; separate business track |

The status labels describe the current stage, not a promise of release dates. I prefer to publish a smaller product that holds up under use than to label unfinished work as production-ready.

## Public project: prerender-audit

Static-site builds produce two related outputs: the pages actually prerendered to HTML and a sitemap advertising the URLs. When these disagree, a route can appear in the sitemap despite having no generated page behind it.

`prerender-audit` compares those outputs before an artifact is promoted. It reports missing advertised pages, unadvertised prerendered pages, duplicate entries, and malformed or off-origin URLs. An empty sitemap or empty HTML output is **not** accepted as an all-clear.

- **Runtime:** Node.js 18+
- **Dependencies:** none; uses the Node.js standard library
- **Tests:** Node's built-in test runner
- **License:** MIT
- **Repository:** [RupamDotInfo/prerender-audit](https://github.com/RupamDotInfo/prerender-audit)

The tool is intentionally scoped. It checks files rather than making live HTTP requests, supports flat HTML output paths, does not recursively fetch sitemap indexes, and does not validate the contents of each HTML page. Those limitations are documented rather than hidden.

## Engineering case studies

The following cases describe engineering work in private projects. The associated source code is not public, so these are presented as engineering summaries rather than as independently browsable code samples.

### 1. A silent boundary between content and routing

A content change caused the sitemap generator to advertise two routes that the static build did not prerender. The cause was a sanitizing function that rebuilt a data object field by field and silently omitted a newly added field. Type checking passed because the declared type was correct; the runtime boundary was not.

**What changed:** fixed the mapping, added a regression test against the real loader rather than mocking away the faulty function, and verified the test failed before the fix and passed afterward. The generalized pre-promotion check became `prerender-audit`.

**The tool then failed its own first CI run, in the way that mattered.** Unit tests import the source module and passed, but a package-level check that packs the tarball, installs it and runs the *installed* command returned success on a deliberately broken build. The cause was the guard that stops the CLI executing when imported as a library: it compared the module name against `process.argv[1]`, which never matches on Linux or macOS, where npm installs a bin entry as a symlink. The command did nothing and exited 0.

That is the worst available failure for a safety check — a CI step running it would have passed on every run, including against broken builds, which is worse than having no check. It is now covered by a test that executes the CLI through a symlink, and the reason for that package-level test is precisely that unit tests alone would never have seen it.

### 2. A CI pipeline that found defects immediately

A project had an extensive test suite but no automated CI. The first run exposed a mismatch between the declared PHP constraint and locked dependency requirements. After that was fixed, the next run exposed tests coupled to deployment-host state. The portable test design was corrected rather than weakening the assertions.

**Lesson:** a suite that passes only on the machines that already work is not a reliable safety net. The pipeline was verified green after the fixes.

### 3. Correcting a false-positive security finding

A scanner flagged a test-only sentinel as though it were a live credential. I initially treated the finding as a production blocker, then traced the evidence and found errors in placeholder detection and fingerprint calculation.

**What changed:** documented the correction, avoided an unjustified production-secret rotation, and strengthened the scanner using synthetic secret patterns. A security finding is a reason to investigate; the evidence should determine the response.

### 4. Publishing with revision verification

A private software catalogue separates saving content from explicitly publishing it. The publishing path records the intended revision and checks what is actually served before accepting promotion. A mismatch prevents the new artifact from being treated as successfully published and keeps a rollback path available.

**Engineering focus:** explicit state transitions, verifiable deployment outcomes, regression tests for the publish boundary, and failure behavior that is exercised rather than merely described.

## Technologies used in my work

- **Systems and desktop:** Rust, Tauri 2, asynchronous programming, SQLite
- **Web and backend:** PHP, Laravel, Vue 3, TypeScript, JavaScript, Vite
- **Mobile:** Kotlin, Jetpack Compose
- **Quality:** automated tests, regression testing, static checks, CI with GitHub Actions
- **Delivery:** Linux, nginx, systemd, content-driven static generation, revision-verified publishing

These are technologies used in current or completed projects, not a claim of equal expertise in every item.

## What I value

- Tests that exercise the real failure boundary.
- Errors that fail clearly instead of producing false success.
- Small, reviewable changes with reproducible verification.
- Honest documentation of what a tool does not support.
- Product decisions guided by reliability and real user needs.

## Links

- **Website:** [rupam.info](https://rupam.info)
- **Software portfolio:** [software.acadefy.org](https://software.acadefy.org)
- **Public source:** [prerender-audit](https://github.com/RupamDotInfo/prerender-audit)
- **X:** [@RupamDotInfo](https://twitter.com/RupamDotInfo)

---

I would rather show a smaller body of work that withstands scrutiny than a longer list of claims that does not.
