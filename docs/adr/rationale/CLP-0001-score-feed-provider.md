# CricketData.org (CricAPI) is the Score Feed — rationale

## Context
The product shows live scores for men's international cricket between the 12 ICC Full Members. `Technical-Context.MD` makes a licensed feed the only permitted source of match data and rules out scraping, so a commercial cricket-data API had to be chosen before any Feature could be specified.

## Considered options
- **CricketData.org (CricAPI)** — REST API with a free tier, which makes it the cheapest way to start a proof of concept. Chosen.
- **Sportmonks Cricket API** — commercial REST API with fixtures, live scores and ball-by-ball data; paid plans only.
- **Roanuz Cricket API** — commercial provider focused on international and major-league cricket; paid.

## Consequences
- Every Score Feed call, and the recorded responses the Tier-1 tests replay through `msw`, are shaped by CricAPI's response format.
- How often the app can refresh a Live Match is bounded by the CricAPI plan's request quota; the PRD sets the refresh interval and the Stale Score threshold against that quota.

## Reversibility
Moderate. Switching provider means rewriting every Score Feed call and re-recording every Tier-1 fixture. Keeping CricAPI's response shapes behind one module, mapped onto the glossary's terms, keeps the rest of the code independent of the provider.

## Unbuilt intent
None.

## Amendment history
- 2026-10-06 — created during the `/factory-grill-with-docs` session that followed the project bootstrap.

## Open checks
- CricAPI's free-tier request quota, and whether its terms permit this use, were not verified when the decision was made. Confirm both before the first Feature that calls the Score Feed is specified.
