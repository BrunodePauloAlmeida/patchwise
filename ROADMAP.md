# Roadmap

**Where this is going:** a patch analytics site I'd trust: every number traceable to matches, honest about small samples, and fed by a crawler that never breaks Riot's rules.

## Constraints that shape this plan (November 2026)

- **Built and tested offline.** The Riot API sits behind a local fake that serves made-up matches in Riot's format and enforces the same rate limits ([Riot API notes](docs/riot-api.md)). Real runs happen later on my machine with my own key.
- **Development keys are tight:** 20 requests per second, 100 per 2 minutes, and they expire every day. The crawler is designed around that budget.
- **Made-up data only:** champions, items and players are generated. No Riot data files, names or images in the repo.
- **Statistics, not machine learning.**

## Phase 1: Synthetic world and fake Riot API

- [ ] Generator: made-up champions (roles, hidden strength per patch, lane matchups), items, and players with hidden skill on a ranked ladder.
- [ ] Matches and timelines in Match-v5 shape, reproducible from a seed.
- [ ] Fake API (League-v4, Account-v1, Match-v5) with the development key limits, the rate-limit headers, 429 with `Retry-After`, and occasional 5xx errors.

## Phase 2: Crawler

- [ ] Ladder → players → match ids → matches and timelines, per region.
- [ ] Rate limiter driven by the response headers; backoff on 429 and 5xx.
- [ ] Checkpoints: killed at any point, it resumes without losing or duplicating a match.

## Phase 3: Warehouse

- [ ] PostgreSQL schema: raw responses plus participants, teams, bans and timeline events, partitioned by patch.
- [ ] Idempotent ingestion.
- [ ] Rollups per patch, champion, role and rank that update incrementally and always equal a full recompute.

## Phase 4: Statistics

- [ ] Win, pick and ban rates with confidence intervals.
- [ ] Shrinkage for small samples, and a minimum sample before a stat is shown.
- [ ] Lane matchups, build paths from purchase events, gold difference at 15 minutes.
- [ ] Patch-to-patch changes, flagged only when they're statistically real.
- [ ] Every statistic checked against the generator's hidden truth.

## Phase 5: Web pages

- [ ] Tier list with filters: patch, role, rank.
- [ ] Champion page: builds, matchup heatmap, win rate by game length.
- [ ] Player page: match history.
- [ ] Patch comparison.

## Done means

- The crawler never exceeds the limits in tests, and survives `kill -9` without duplicates.
- Incremental rollups equal a full recompute.
- On synthetic data, 95% intervals contain the true win rate about 95% of the time.
- Tier list pages answer in under 300 ms with 1 million matches.

## Open decisions

- Which ranks and regions to crawl first.
- What to do when `teamPosition` is empty.
- How long to keep raw responses.
- The minimum sample before a champion appears in the tier list.

## Later

- Real Riot data on my machine, with a development key.
- More queues (Flex, ARAM).
