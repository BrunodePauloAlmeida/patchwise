# Riot API notes

What Patchwise needs from the Riot Games API. Checked against the developer portal in November 2026. Development and tests run offline against a fake API that copies these rules; recheck the details with a real key before going live.

## Keys and limits

- Keys come from the [developer portal](https://developer.riotgames.com/). A development key is for prototypes and deactivates every 24 hours. A public product needs a registered project and a production key.
- Development and personal keys: 20 requests every 1 second and 100 requests every 2 minutes.
- Production keys start at 500 requests every 10 seconds and 30,000 every 10 minutes.
- Limits are enforced per region: one key can use its full limit on two regions at the same time.
- Besides the app limit, each method has its own limit. Responses carry `X-App-Rate-Limit`, `X-App-Rate-Limit-Count`, `X-Method-Rate-Limit` and `X-Method-Rate-Limit-Count`, written as `limit:seconds` pairs (for example `20:1,100:120`). A 429 response comes with `Retry-After`.
- Send the key in the `X-Riot-Token` header.

## Routing

- Platform hosts (`https://br1.api.riotgames.com`, `na1`, `euw1`, `kr`, ...) serve League-v4 and Summoner-v4.
- Regional hosts (`https://americas.api.riotgames.com`, `europe`, `asia`, `sea`) serve Account-v1 and Match-v5.
- For Match-v5, `americas` serves NA, BR, LAN and LAS; `asia` serves KR and JP; `europe` serves EUNE, EUW, ME1, TR and RU; `sea` serves OCE, SG2, TW2 and VN2.

## Endpoints Patchwise uses

| Purpose | Endpoint | Host |
| --- | --- | --- |
| Top of the ladder | `/lol/league/v4/challengerleagues/by-queue/{queue}` (also `grandmasterleagues`, `masterleagues`) | platform |
| Ladder by tier | `/lol/league/v4/entries/{queue}/{tier}/{division}?page={n}` | platform |
| A player's ranks | `/lol/league/v4/entries/by-puuid/{puuid}` | platform |
| Riot ID to PUUID | `/riot/account/v1/accounts/by-riot-id/{gameName}/{tagLine}` | regional |
| A player's match ids | `/lol/match/v5/matches/by-puuid/{puuid}/ids?queue=420&start=0&count=100` | regional |
| One match | `/lol/match/v5/matches/{matchId}` | regional |
| Its timeline | `/lol/match/v5/matches/{matchId}/timeline` | regional |

Ranked solo queue is `RANKED_SOLO_5x5` in League-v4 and queue id `420` in Match-v5. Match ids carry the platform, as in `BR1_2934817265`.

## Fields Patchwise reads

Names as they appear in Match-v5 responses. Confirm them on a real response before relying on them.

- `metadata.matchId` and `metadata.participants` (PUUIDs).
- `info.gameVersion` (the game build; its first two numbers identify the patch), `info.gameDuration` (seconds), `info.queueId`, `info.gameCreation`.
- `info.participants[]`: `puuid`, `championId`, `championName`, `teamId`, `teamPosition` (`TOP`, `JUNGLE`, `MIDDLE`, `BOTTOM`, `UTILITY`, sometimes empty), `win`, `kills`, `deaths`, `assists`, `goldEarned`, `item0` to `item6`.
- `info.teams[]`: `teamId`, `win`, `bans[]` with `championId` and `pickTurn`.
- Timeline: `info.frames[]`, one per minute, each with `participantFrames` (`totalGold`, `xp`, `position`) and `events` (`ITEM_PURCHASED`, `CHAMPION_KILL`, `ELITE_MONSTER_KILL`, `BUILDING_KILL`, ...).

## Before publishing anything

Read Riot's developer policies: a public site needs a registered project, and it must say that it isn't endorsed by Riot Games. Champion and item names and images come from Riot's static data on a machine with internet; none of it goes into this repo.
