# Rupam Bhaduri

**Software Engineer · Product Builder · AI & EdTech**

I build software products end to end — from the CMS and publishing pipeline
behind them, through the APIs, to the accessible front end. My focus is
engineering that is verifiable: a claim that something works should come with a
test, a measurement, or a check that runs in CI, and a limitation that is stated
rather than implied.

I am CEO and Founder at **AcadeFy**, where I work across the platform and on
the developer tooling that ships it.

---

## About

The problems I gravitate toward are the unglamorous ones that decide whether a
product actually works:

- **Content that cannot silently drift from what is served.** A CMS where an
  edit either reaches the public site exactly as reviewed, or does not reach it
  at all.
- **Deployments that cannot quietly ship a broken page.** Publishing pipelines
  with real verification and a rollback that is actually exercised.
- **Checks that fail when they should.** A green pipeline that has never been
  shown to catch anything is indistinguishable from no pipeline.
- **Accessibility and correctness in the same pass**, not as a later phase.

I care about the failure modes more than the happy path. Most of the work
below is about a system refusing to do the wrong thing.

---

## Selected work

| Project | What it is | Status |
|---|---|---|
| [**prerender-audit**](https://github.com/RupamDotInfo/prerender-audit) | Zero-dependency CLI that checks a static site's sitemap and prerendered output agree | Public, released |
| **AcadeFy** | Laravel platform and multi-tenant product infrastructure | Private |
| **AcadeFy Software** | CMS-driven static software catalogue with revision-verified publishing | Private, in production |

The private work is not open source, but the engineering behind it is described
below and every claim is traceable to a commit or a test.

---

## Engineering case studies

Each of these describes a real problem, what made it hard, what I chose, and
what verified it. Where something remains unresolved, it is stated.

### 1. A silent filter between the CMS and the router

**The problem.** I added a new top-level collection to the public content
payload that the static build reads. The build advertised the new URLs in its
sitemap and prerendered none of them. Two advertised URLs would have returned
404.

**Why it was hard to see.** Two components read the *same* source data through
different paths. The sitemap generator read the raw payload. The router read
the same data through a sanitising function that rebuilt the object key by key.
A key had been added to the payload and to the TypeScript type — but not to the
sanitiser, which dropped it in silence. Type checking passed, because the type
was correct. The sanitiser was not part of the type boundary.

**What caught it.** The deployment's post-promotion verification requested every
sitemap URL and found two returning 404. It refused to promote the artifact and
restored the previous release. The site was never broken.

**The part I'm not proud of.** The test I wrote alongside the feature passed
throughout. It mocked the loader wholesale — which meant it mocked away the
exact function that was broken. A test that mocks the thing most likely to be
wrong proves nothing about it.

**The fix.** One line in the sanitiser, plus a new test file that calls the
*real* loader against a stubbed CMS response. I verified the new test fails
against the pre-fix code and passes after, so it actually pins the behaviour
rather than merely agreeing with it.

**Trade-off.** Detection happened after promotion, at the URL-probe stage, not
during the build. That is a late place to find it. The underlying generalisable
lesson — that a sitemap and its prerendered output are two independent outputs
with nothing forcing them to agree — became
[`prerender-audit`](https://github.com/RupamDotInfo/prerender-audit), which
checks the agreement before an artifact is promoted.

### 2. Correcting a security finding that was wrong

**The problem.** An audit flagged a credential-shaped literal committed in a
deployment script and recommended rotating the production secret before the
repository was ever pushed. I reported it as a genuine blocker.

**Why the original finding was wrong.** The literal was the self-evidently
invalid string `definitely-not-the-secret`, present so the smoke tests could
assert that the deploy callback *rejects* wrong secrets. Three distinct values
were in play and only one was live:

| | Length | Accepted by production? |
|---|---|---|
| Committed literal | 25 | No — HTTP 401 |
| Second script copy | 11 | No — HTTP 401 |
| Live production secret | 64 | Yes — HTTP 200 |

Two measurement errors compounded it. The placeholder check tested a short word
list that did not contain these strings, so it reported them as non-placeholders.
A later fingerprint hashed the entire regex match including the header prefix
rather than the captured value, turning a 25-character string into 42 and making
it look opaque.

**What I did.** Verified by fingerprint comparison and by actually calling the
endpoint, not by re-asserting. Found no rotation was warranted — rotating a
working production secret to cover a non-secret would have disrupted a live
publishing pipeline for no benefit. Recorded the correction in the project's own
known-issues document rather than deleting it.

**Then.** Added a dependency-free secret scanner to the repository, with the
known-safe values allowlisted explicitly rather than by disabling the rule. It
reports file, line and rule and never echoes a matched value. I proved it works
by planting six secret shapes and confirming all six are caught.

**The lesson I keep.** A finding that would change production behaviour has to
be proven before it acts. "It looks like a secret" is a reason to investigate, not
a reason to rotate.

### 3. CI that found two real defects on its first runs

**The problem.** A repository with 325 backend tests, 90 frontend tests and a
type check had no CI at all. Nothing ran automatically. The first push of 29
commits had no automated verification behind it.

**Run 1 failed on dependency resolution**, not on a test. The workflow used PHP
8.3 because `composer.json` declared `"php": "^8.3"` — but seventeen locked
packages required `>=8.4.1`. The declared constraint was simply wrong, so no PHP
8.3 machine could ever have installed the backend. Corrected to `^8.4`; CI moved
to PHP 8.5 to match production.

**Run 2 installed cleanly and then failed 4 of 325 tests.** Every one of them
was coupled to state that exists only on the deploy host: the publication state
directory, and the application's own SQLite file. The suite passed on my
machine and on production and could not run anywhere else — precisely backwards
for a suite that most needs to run in CI.

The fix had its own trap. My first attempt pointed the database guard test at a
temporary file, which would have made it **assert nothing while passing** — the
guard deliberately ignores any path that is not the real database. I caught it,
extracted the decision into a pure predicate, and tested that directly with
scratch files. The earlier weaker version was reverted rather than shipped.

**Run 3: all three jobs green.** Two genuine defects found in the first two
executions of a pipeline that had never run.

### 4. Publishing that refuses to ship something wrong

The catalogue publishes through a CMS rather than SQL or a seed script, and
separates saving from publishing: an edit is stored privately, reviewed, and
only an explicit publish control rebuilds the public site. That separation is
enforced by tests that assert saving dispatches **zero** deployment runs, and
that the publish control dispatches exactly one.

Every deployment is recorded with its intended revision, the revision verified
at the origin, and the revision in the live manifest. Promotion is conditional
on those agreeing. When they disagree, the artifact is not promoted and the
previous release is restored — a path that is exercised regularly enough to be
trusted rather than theoretical.

---

## Technology

Used in the work described here, not aspirationally:

**Backend** — PHP 8.4+, Laravel, SQLite, Eloquent, Filament, Livewire,
PHPUnit/Pest, Composer

**Frontend** — Vue 3, TypeScript, Vite, `vite-ssg`, Tailwind CSS, Vitest

**Platform** — Linux, nginx, systemd, Cloudflare, GitHub Actions

**Practices** — content-driven static generation, revision-verified
publishing with rollback, WCAG 2.1 AA accessibility, contract-shaped regression
tests, secret scanning in CI

---

## Current focus

- Extending the development journal on the software catalogue so in-progress
  work is visible without overclaiming readiness
- Reducing the time between a content edit and a verified, published change
- Turning the incidental deploy scripts in that project into one documented path

---

## Links

- **Website** — [rupam.info](https://rupam.info)
- **Software portfolio** — [software.acadefy.org](https://software.acadefy.org)
- **Public code** — [github.com/RupamDotInfo/prerender-audit](https://github.com/RupamDotInfo/prerender-audit)
- **X / Twitter** — [@RupamDotInfo](https://twitter.com/RupamDotInfo)

---

## A note on claims

Every technical claim above is traceable to a commit, a test, or a measurement
in the private projects. Where I could not verify something, I have said so —
including the two things I had to correct in my own audit, and the tooling gap
that remains.

I would rather show a smaller body of work that holds up under inspection than
a longer list that does not.