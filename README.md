<div align="center"><img src="cover.png" width="100%"></div>

# Viral Script Generator

> Turns a creator's own back catalogue of viral short-form videos into a private model of *why* they went viral — then writes ten ranked, ready-to-record scripts on demand, in that creator's exact voice.

<p align="center"></p>

<p align="center">
  <img src="https://img.shields.io/badge/Next.js_16-000?logo=nextdotjs&logoColor=white">
  <img src="https://img.shields.io/badge/Supabase_·_pgvector-3ECF8E?logo=supabase&logoColor=white">
  <img src="https://img.shields.io/badge/Deepgram_Nova--3-13EF93?logo=deepgram&logoColor=black">
  <img src="https://img.shields.io/badge/Gemini_2.5_Pro-8E75FF?logo=googlegemini&logoColor=white">
  <img src="https://img.shields.io/badge/Apify-FF9012?logo=apify&logoColor=white">
  <img src="https://img.shields.io/badge/Cohere_Rerank-39594D?logo=cohere&logoColor=white">
</p>

## The problem

A creator posting daily short-form video lives or dies on the first three seconds. Their best hooks are already sitting in their own feed — but that knowledge is trapped in a few hundred past videos nobody has the time to reverse-engineer. Generic "write me a TikTok script" prompting ignores all of it and produces off-brand filler. The goal here was the opposite: **learn only from what has already worked for this exact account, and never drift from that voice.**

## What it does

1. **Ingests the back catalogue.** Scrapes ~4,300 reels for an account, computes a `viral_score` (view-outlier, engagement-rate and share-rate against a rolling 30-day median), filters out reposts/collabs/pinned, and keeps the **top 500** as the training set.
2. **Turns video into structured knowledge.** Downloads each winner, transcribes it with **Deepgram Nova-3**, then a vision-capable analysis model extracts the hook, narrative arc, pillar, power phrases and — critically — separates the *idea* from the *wording* so the two can be recombined later.
3. **Builds the moat, once.** A meta-analysis over the top 100 produces an approved **voice profile** and a taxonomy of hook types and arcs. Everything is embedded (hook + body vectors) into **pgvector** for hybrid retrieval.
4. **Generates on demand.** Pick a content pillar → hybrid retrieval pulls the 12 most relevant proven scripts → a **generator** drafts 20 candidates at high temperature → a **critic** scores and ranks them at low temperature → the **top 10** come back, each 50–90 words (≈20–30s of voiceover) with a per-field score.

## How it holds up in production

- **Deterministic where it must be.** Scoring, tiering, de-duplication and the 50–90-word length gate are plain code with unit tests — the model never decides what counts as "viral."
- **Two-phase, not one-shot.** Splitting generation (creative, temp 0.7) from criticism (strict, temp 0.1) is measurably more reliable than asking one model for ten finished scripts.
- **Multi-tenant by construction.** Every row carries `workspace_id` + `client_id`, enforced by Postgres RLS; one creator's library can never leak into another's retrieval.
- **A real feedback loop.** Editors mark each script used / edited / rejected; once posted, the video's actual views flow back as a `proven_lift` signal that re-weights retrieval — the system gets sharper the more it's used.

## Results

| | |
|---|---|
| **~4,300 → 500** | reels scraped, scored and distilled to the proven training set per account |
| **10 ranked scripts in < 90s** | from one pillar click, each length-validated and critic-scored |
| **≥ 6 of 10 hook types** | represented across a single batch — diversity is enforced, not hoped for |

## Stack

- **App** — Next.js 16 (App Router), TypeScript, Tailwind, magic-link auth with an authorized-user allowlist
- **Data** — Supabase Postgres + **pgvector**, Supabase Storage for raw media, row-level security throughout
- **Ingestion** — Apify (Instagram) → Deepgram **Nova-3** transcription → Gemini **2.5 Pro** pattern extraction → 768-dim embeddings
- **Retrieval + generation** — hybrid pgvector + structured filters, Cohere rerank, a two-phase generator/critic reasoning pipeline
- **Ops** — queue-backed ingestion workers, a weekly incremental re-scrape, health checks on every dependency

## Try it live

**** — access is allowlisted (real creator data sits behind it), so the live URL opens on the sign-in screen.

---

*Case study. Full source and the analysed content library are private and available under NDA.*
