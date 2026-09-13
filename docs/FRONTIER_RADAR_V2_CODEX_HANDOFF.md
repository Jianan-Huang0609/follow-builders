# Frontier Radar v2 — Personal AI Intelligence OS

## Context

This repository already implements the core philosophy: **Follow Builders, Not Influencers** — tracking builders, podcasts, X posts and company blogs and producing a curated digest.

The next stage should evolve it from a content digest into a **personal AI intelligence system** that connects Podcast + YouTube + Papers + X into one cross-source research/learning loop.

The objective is not to collect more content. The product should answer:

> What changed in the areas I care about, why does it matter, and what should I actually spend time reading / watching / listening to?

The primary success metric is **useful insights per minute of user attention**, not number of items collected.

---

## Product model

Four source types:

1. **Podcast** — operator experience, business transformation, market judgment
2. **YouTube** — demos, long-form interviews, builder explanations
3. **Papers** — evidence, methods, benchmarks, research frontier
4. **X / Twitter** — weak signals, real-time opinions, emerging topics

These should not be rendered as four independent feeds. The system should identify when the same underlying theme is appearing across multiple sources and combine them into one **Topic Cluster / Signal**.

Example:

X: several builders start discussing long-running coding agents
→ YouTube: builder interview explains architecture
→ Paper: related agent-memory / long-horizon evaluation paper
→ Podcast: investor / enterprise discussion asks whether it becomes a product category
→ Radar output: one consolidated signal with evidence from all four sources.

---

## 1. Follow Registry

The Follow List should become a first-class asset and be independent of platforms.

Support entities such as:

- person
- company
- research lab
- podcast
- YouTube channel
- X account
- X List
- researcher
- paper
- research topic

Roles:

- builder
- founder
- company
- investor
- researcher
- creator

Suggested fields:

```yaml
name:
entity_type:
roles: []
topics: []
priority:
active:
sources:
  podcast:
  youtube:
  x:
  papers:
```

Example:

```yaml
name: Ethan Mollick
entity_type: person
roles:
  - researcher
  - creator
topics:
  - AI adoption
  - enterprise AI
  - future of work
sources:
  x: ...
  youtube: ...
  papers: ...
```

Prefer a human-editable YAML or JSON registry.

---

## 2. Source Connectors

### Podcast

Use RSS as the primary update mechanism. Podcast Index may be used for discovery / metadata.

Do not make transcription a hard dependency for V1. Use existing transcripts where available and fall back to metadata / show notes.

### YouTube

Use YouTube Data API / upload playlists for discovery.

Retrieve title, description, publish time, channel and transcript/captions where accessible.

`yt-dlp` may be used as an auxiliary extractor, not as the main discovery layer.

### Papers

Use Semantic Scholar, OpenAlex and/or arXiv.

Support:

- author watch
- topic / keyword watch
- seed-paper watch
- citation graph
- related/recommended papers

The goal is not a daily keyword search dump; the paper feed should be connected to the user's existing research graph.

### X

Prefer curated Lists / selected accounts rather than the whole home timeline.

Suggested groups:

- AI Researchers
- AI Founders
- AI Investors
- AI Builders

X can be Phase 2 if API cost / implementation complexity slows the core MVP.

---

## 3. Unified ContentItem schema

Every connector should normalize into one schema.

Suggested fields:

```text
id
source_type
source_id
creator
title
url
published_at
raw_text
transcript
description

topics[]
entities[]
people[]
companies[]

summary
key_claims[]
evidence[]
contrarian_points[]

relevance_score
novelty_score
authority_score
market_signal_score
actionability_score

related_items[]
processing_status
created_at
updated_at
```

Also define a `TopicCluster` / `Signal` schema capable of linking multiple ContentItems that describe the same underlying development.

---

## 4. Intelligence pipeline

Target pipeline:

```text
Source
→ Fetch
→ Normalize
→ Deduplicate
→ Extract text/transcript
→ LLM enrichment
→ Topic classification
→ Entity extraction
→ Embedding
→ Similarity clustering
→ Cross-source signal detection
→ Ranking
→ Digest generation
```

Key product requirement:

If one idea appears as an X thread, a YouTube interview, a paper and a podcast discussion, show it as **one signal with multiple pieces of evidence**, not four unrelated feed items.

---

## 5. Explainable Intelligence Score

Initial scoring proposal:

- Relevance: 30%
- Novelty: 20%
- Authority: 20%
- Cross-source signal: 15%
- Actionability: 15%

Do not rely on one opaque LLM score.

Use deterministic features where possible:

- creator priority
- topic match
- recency
- citation / engagement signals when available
- number of independent sources
- semantic similarity to user interests

The UI/output should be able to explain why an item ranked highly.

---

## 6. Outputs

### Daily Radar

Return at most **5 high-value signals**.

For each signal:

- What happened?
- Why now?
- Who is discussing it?
- Cross-source evidence
- Why it matters to the user
- Recommended content to consume
- What can safely be skipped because it repeats the same idea

The `Skip` recommendation is important. The product should actively reduce information load.

### Weekly Intelligence Review

Generate:

- Top 3 emerging themes
- Important company movements
- Important research developments
- Interesting founder / investor opinions
- Contradictions / disagreements
- One recommended deep dive
- One proposed thesis:

> Based on this week's evidence, what belief about AI should I reconsider or update?

### Monthly Frontier Map

Show:

- topics increasing in importance
- topics declining
- new people / companies worth following
- low-value sources worth unfollowing
- high-value sources discovered
- changes in the user's interest graph

---

## 7. Architecture preference

Reuse the useful parts of the existing `follow-builders` stack rather than rewriting everything.

Potential architecture:

```text
RSS / APIs / existing feed
        ↓
Universal ingestion layer
        ↓
Python processing services
        ↓
PostgreSQL + pgvector
        ↓
LLM enrichment / clustering / synthesis
        ↓
Daily + Weekly outputs
        ↓
Existing newsletter / web layer can become a presentation surface later
```

Miniflux and/or n8n can be considered for feed storage / scheduling / orchestration if they simplify the system, but do not introduce them unless they reduce complexity relative to the existing codebase.

---

## 8. MVP scope

Phase 1 should support:

- YouTube
- Papers
- Podcasts
- ~20–30 followed sources

X can be added after the ranking / clustering pipeline proves useful.

Expected CLI shape:

```bash
radar fetch
radar process
radar daily
radar weekly
radar sources list
radar sources add
radar sources disable
```

Expected artifacts:

```text
daily.md
weekly.md
```

No dashboard in V1.

---

## First implementation task — STOP BEFORE CODING

Before making substantial implementation changes:

1. Inspect the existing `follow-builders` repository architecture and identify what can be reused.
2. Compare the current data/source pipeline with the proposed Frontier Radar v2 pipeline.
3. Propose the repository / module architecture.
4. Define the `FollowRegistry`, `ContentItem`, and `TopicCluster` schemas.
5. Recommend which existing components should be kept, modified, deprecated, or replaced.
6. Propose an MVP implementation sequence.
7. Call out API / cost / reliability risks, especially Podcast transcripts, YouTube transcripts and X.

**Do not implement the full system yet.**

Return the architecture proposal, schema proposal, major decisions/tradeoffs and an implementation plan for user review.

Only proceed to implementation after explicit review/approval.

---

## After approval: first build milestone

Once approved:

1. Implement one YouTube connector.
2. Implement one Semantic Scholar / OpenAlex connector.
3. Reuse or adapt the existing Podcast ingestion path.
4. Ingest ~30 real content items.
5. Demonstrate normalization + deduplication + topic clustering.
6. Generate one real `Daily Radar` example.
7. Show diffs and test results for review.

Do not build the dashboard yet.
