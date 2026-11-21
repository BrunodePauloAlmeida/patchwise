# Patchwise

**League of Legends patch analytics: what wins, in which role, on this patch.**

Patchwise crawls ranked League of Legends matches from the Riot API, stores them in PostgreSQL and turns them into the numbers players look up before a game: win, pick and ban rates per champion, role and patch, lane matchups, item build paths and how games are won over time. It's the kind of site op.gg or u.gg runs, built from the crawler up.

The hard parts sit under the tier list: a crawler that respects Riot's rate limits and survives being killed halfway, a warehouse that updates its numbers as matches arrive, and statistics honest enough to say "not enough games yet" instead of ranking a champion first on 12 matches.

## Status

Early days: the project is set up and planned, and the code comes next. See the [roadmap](ROADMAP.md).

## How it will work

1. **Synthetic world.** A generator creates made-up champions, items and players with hidden strengths, then plays ranked matches between them. A local fake API serves those matches in the shape of the Riot API, with the same rate limits. Because the true strengths are known, every statistic can be checked against the truth.
2. **Crawler.** Starts from the ranked ladder, follows players to their match history and downloads matches and timelines. It reads Riot's rate-limit headers, backs off on 429, and resumes after a crash without fetching or storing anything twice.
3. **Warehouse.** Raw responses and normalized tables in PostgreSQL, partitioned by patch, with rollups that update as each batch lands.
4. **Statistics.** Win, pick and ban rates with confidence intervals; shrinkage for small samples; lane matchups; build paths from purchase events; gold difference at 15 minutes; and what changed from one patch to the next, flagged only when the change is real.
5. **Web pages.** Tier list, champion page (builds, matchup heatmap), player page (match history) and a patch comparison.

Statistics only: no machine learning and no win predictor.

## Built with

Python 3.12, PostgreSQL, httpx for the crawler, FastAPI with Jinja2 and htmx for the pages, pytest for tests. What Patchwise needs from the Riot API is in [docs/riot-api.md](docs/riot-api.md).

## Setup

Nothing runs yet beyond the test suite.

```bash
cp .env.example .env
python -m venv .venv
.venv/bin/pip install -e ".[dev]"
.venv/bin/python -m pytest -q
```

Against the real Riot API you need a key from the [Riot Developer Portal](https://developer.riotgames.com/). Development keys expire every 24 hours.

## Privacy

Development and tests use made-up players only. With real data, Patchwise stores public match data keyed by Riot's player IDs. `.env` and local data are ignored by git.

## License

[MIT](LICENSE). Patchwise isn't endorsed by Riot Games and doesn't reflect the views of Riot Games or anyone officially involved with League of Legends. League of Legends is a trademark of Riot Games, Inc.
