# Agora — demo bundle

This is the real Agora product — the same Django + Astro/Svelte app that
powers client engagements at [agora.fridai.dev](https://agora.fridai.dev) —
pre-loaded with a fully-analyzed sample engagement: 1,643 public Emirates
airline reviews (Trustpilot + Skytrax), already run through the theme,
sentiment, and per-topic-stance pipeline. Browse the resulting dashboard
exactly as a client would see it.

**The AI pipeline code itself is not included in this bundle.** The image
you're pulling ships an inert stub in its place — you're looking at the
finished analysis, not a live re-run of it. See "What's excluded" below.

## Quickstart

Requires Docker (Compose v2) and about 3 GB free (image pulls + a small
Postgres volume). No API keys, no signup.

```bash
git clone https://github.com/fridai-dot-dev/agora-demo.git
cd agora-demo
docker compose up -d
```

First run takes several minutes, not a minute or two — most of it is pulling
the two images (~3.4 GB combined), then Postgres restores the seed data and
Django runs migrations. Measured clean-machine wall clock (`git clone` to a
working login, cold local image cache): **~4–6 minutes** on a normal
connection; a slower line or a host with none of the shared base layers
already cached may take longer. Watch it with `docker compose logs -f backend`
if you want to see progress; `docker compose ps` shows `healthy` once it's
ready. Subsequent restarts (`docker compose up -d` again, images already
local) take well under a minute.

Then open **http://localhost:3000** and sign in:

| Username | Password |
|---|---|
| `demo` | `agora-demo` |

That's it — three commands and a login. To stop it: `docker compose down`
(add `-v` to also drop the database volume and start fully fresh next time).

## Architecture

```mermaid
flowchart LR
    You(["You — localhost:3000"]) --> Frontend[frontend<br/>Astro + Svelte]
    Frontend --> Backend[backend<br/>Django + DRF]
    Backend --> Postgres[(postgres<br/>seeded with Emirates data)]
    Backend --> Redis[(redis<br/>cache + sessions)]
    Worker["worker <br/><i>not included</i><br/>(no new analysis runs here)"]
    style Worker fill:#eee,stroke:#999,stroke-dasharray: 5 5,color:#999
    Backend -.->|"would queue jobs to"| Worker
```

Four services boot: `frontend`, `backend`, `postgres`, `redis`. The greyed
`worker` box is what a real engagement also runs (an RQ worker that executes
the analysis pipeline as a background job) — it's intentionally absent here
because this bundle ships no LLM API key and no pipeline code, so there is
nothing for a worker to do. The dashboard you're browsing was analyzed
once, in a real engagement, and the result was captured into this bundle's
seed data.

## What's excluded (and why)

- **The theme-analysis pipeline** (`consultation_analyser/themefinder/` in
  the real repo) — the prompts, LLM orchestration, and clustering logic that
  turn raw verbatims into themes. Replaced with an inert stub of the same
  module shape, so the app boots and serves already-computed results without
  being able to run new analysis.
- **Any API key or credential that could reach a live system** — no
  `GOOGLE_API_KEY`, no SMTP, no Sentry, no AWS. The `demo` login works
  because email-based MFA is off by default on a fresh deployment (a client
  engagement opts back in via its own config); there's no outbound email in
  this bundle at all.
- **A background worker.** See above — nothing here queues pipeline jobs.
- **Write access that matters.** You can click around the review UI, but
  there's no LLM behind it to act on anything you change, and there's no
  path out to a real inbox, bucket, or third-party API.

## Want the full thing?

This is a sample of what a real client engagement looks like once analysis
is done. For a live pipeline against your own data, or a walkthrough of the
full analysis workflow, see [agora.fridai.dev](https://agora.fridai.dev) or
get in touch.
