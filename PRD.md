# PRD — Cricket Live

## Problem Statement

A cricket follower who wants to know what is happening in men's international cricket right now has to dig through general sports sites, which mix international cricket with domestic leagues, franchise tournaments, women's, A and age-group cricket, and news. Seeing at a glance which International Matches between the major national teams are live, and the Score of each, takes several clicks and a lot of noise. Those sites also rarely say how old a Score is, so a viewer cannot tell a current Score from a Stale Score.

## Solution

A single web page that lists every Live Match between two Full-Member Teams, in any Format (Test, ODI, T20I), with a one-line Score for each that shows when it was Last Updated. When a match ends it stays on the page as a Recently Completed Match with its Result for a short window, then drops off. When the Score Feed is late or down, the page says so plainly, and never presents a Stale Score as live. Nothing else appears: no Fixtures, no Scorecards, no news, no accounts.

## Requirements

### Seeing what is live

1. As a cricket follower, I want one page that lists every Live Match, so that I can see everything happening in international cricket right now in one place.
2. As a cricket follower, I want the page to show only International Matches, meaning men's Tests, ODIs and T20Is where both teams are Full-Member Teams, so that domestic, franchise, women's, A-team and Associate matches do not crowd the list.
3. As a cricket follower, I want a match to appear on the page from the toss, so that I can follow it from the start.
4. As a cricket follower, I want a Live Match to stay on the page through every break in play (drinks, lunch, tea, rain or bad-light delays, the interval between innings, and overnight in a Test), so that a pause in play does not make a match disappear.
5. As a cricket follower, I want each match's two teams named clearly, so that I can find the match I care about quickly.
6. As a cricket follower, I want each match labelled with its Format, so that I know whether a Score is from a Test, an ODI or a T20I.
7. As a cricket follower, I want a plain "No live matches right now" message when nothing is live, so that an empty page does not look broken.
8. As a cricket follower, I want Fixtures that have not reached the toss kept off the page, so that the page shows only what is happening now.
9. As a cricket follower watching several matches, I want Live Matches in a stable, predictable order, so that a match does not jump around the list each time the page refreshes.

### The Score

10. As a cricket follower, I want each match's one-line Score to show the batting side's runs/wickets and overs bowled in the current Innings, so that I can see the state of play at a glance.
11. As a cricket follower, I want the Score of an ODI or T20I chase to show the Target, so that I know how many runs the chasing side needs.
12. As a cricket follower, I want a Target revised under the DLS method marked as revised, with the overs it applies to, so that I am not misled after a rain interruption.
13. As a cricket follower, I want a Test match's Score to show every Innings total so far, joined with "&", so that I can follow the whole match and not just the current Innings.
14. As a cricket follower, I want a Test match's Score to show the lead or deficit, so that I know where the match stands.
15. As a cricket follower, I want a Test match's Score to show the day of the match, so that I know how much time is left.
16. As a cricket follower, I want the Score to read the same way across matches of the same Format, so that I can compare matches without re-reading.
17. As a cricket follower, I want a match that has had its toss but no ball bowled yet to show that it is live and yet to start, so that I know the match is on.

### Keeping it current

18. As a cricket follower, I want the page to refresh Scores on its own while it is open, so that I do not have to reload it to see the latest Score.
19. As a cricket follower, I want every Score to show its Last Updated time, so that I can judge how current it is.
20. As a cricket follower, I want a Score that has gone too long without an update marked as a Stale Score and not shown as live, so that I never mistake old data for the current state of play.
21. As a cricket follower, I want a plain banner when the Score Feed is delayed or unavailable (a Feed Delay), so that I know the app's information is behind, not that the match has stopped.
22. As a cricket follower, I want the page to recover on its own when the Score Feed comes back, so that I do not have to reload after a Feed Delay.
23. As a cricket follower, I want a Live Match to stay on the page during a Feed Delay with its Last Updated time, so that the match does not vanish because the data paused.

### When a match ends

24. As a cricket follower, I want a match that has just finished to stay on the page as a Recently Completed Match with its Result, so that if I check just after the finish I still see how it ended.
25. As a cricket follower, I want the Result to state the winner and the margin in runs or wickets, or say tie, draw, no result or abandoned, so that the outcome is unambiguous.
26. As a cricket follower, I want a Recently Completed Match clearly marked as finished, so that I do not mistake it for a Live Match.
27. As a cricket follower, I want a Recently Completed Match to leave the page once its window closes, so that the page stays focused on what is live.

### Tone and presentation

28. As a cricket follower, I want short, factual, scoreboard-style wording, so that I can read the page in a glance.
29. As a cricket follower on a phone, I want the page to work at phone width, so that I can check Scores on the go.
30. As a cricket follower, I want the page to open quickly, so that checking a Score takes seconds.

### Operating the POC

31. As the product owner, I want all match data to come only from the licensed Score Feed, so that the product never relies on scraped data or breaches another site's terms.
32. As the product owner, I want the number of Score Feed requests to stay the same however many people have the page open, so that viewer numbers cannot exhaust the CricAPI request quota.
33. As the product owner, I want every Score Feed call and every Feed Delay logged, so that I can see the feed's reliability and quota use.
34. As the product owner, I want the CricAPI key never exposed to the browser, so that it cannot be copied and abused.
35. As the product owner, I want a malformed or unexpected Score Feed response rejected and treated as a Feed Delay, so that the page never shows a garbled Score.

## Implementation Decisions

- **Next.js with TypeScript, hosted on a single Azure App Service `poc` environment.** Set in `Technical-Context.MD`. One app serves both the page and the server-side calls to the Score Feed.
- **CricketData.org (CricAPI) is the sole Score Feed** (ADR CLP-0001). Every Fixture, Score and Result comes from it, and nothing is scraped.
- **The Score Feed is called only from the server, never from the browser.** This keeps the CricAPI key out of the browser (requirement 34).
- **The server shares one Score Feed result among all viewers.** It calls the Score Feed at most once per refresh interval and serves every viewer from that result, so request volume does not grow with viewers (requirement 32).
- **The browser polls our own server on a fixed refresh interval.** It never polls CricAPI. The page updates in place, with no reload.
- **Our server decides whether a match is an International Match and what its state is**, using the Context.MD definitions: men's Test, ODI or T20I between two Full-Member Teams; Live Match from toss to Result; Recently Completed Match during the post-Result window. The browser gets only what it shows.
- **The 12 Full-Member Teams are a fixed list owned by the product.** CricAPI's own team categorisation is not used. Changing the list (if the ICC admits or removes a Full Member) is a deliberate change.
- **Every response from the Score Feed is checked before use.** A response that is malformed or missing required fields is treated as a Feed Delay, never shown (requirement 35, the fail-closed principle).
- **Freshness is judged against Last Updated.** A Score past the staleness threshold is a Stale Score and is never presented as live; failing to reach the Score Feed, or getting no fresh data from it, is a Feed Delay.
- **No database and no user accounts.** The page holds no state beyond the latest shared Score Feed result.
- **Logging is structured JSON to stdout.** Every Score Feed call is logged with its status, latency and remaining quota where reported, and so is every Feed Delay, as `Technical-Context.MD` requires.

## Testing Decisions

- **A good test checks what a viewer can see, not how the code does it.** Tests assert on the rendered page: which matches are listed, the Score text, the Last Updated time, the Result, and the banners. They never assert on internal functions or data shapes.
- **Only the Live page is tested, end to end, with Playwright.** No module is unit-tested in isolation. Each test runs the whole app, from the page through our server to the Score Feed boundary.
- **The Score Feed is faked at its boundary with recorded CricAPI responses (MSW).** This is the Tier-1 recorded replay in `Technical-Context.MD`'s testing standard, and it gives the CricAPI seam a test with real IO on our side.
- **Scenarios the recorded responses must cover:**
  - no Live Match
  - one Live Match in each Format
  - a Test on day 3 with a lead
  - a chase with a DLS-revised Target
  - a match during a rain delay
  - a match between toss and first ball
  - a Recently Completed Match for each kind of Result
  - an Associate, women's and franchise match that must be filtered out
  - a Stale Score
  - the Score Feed unavailable
  - a malformed response
  - the Score Feed recovering after a delay
- **Tier 2 (live CricAPI) runs on a schedule, within the request quota**, as the testing standard requires. It checks that recorded responses still match the real feed's shape.
- **Prior art:** none, because the repo has no code yet. The first Feature sets the pattern.

## Out of Scope

- Fixtures (matches before the toss), schedules and series pages
- Scorecards (per-batter and per-bowler detail), ball-by-ball commentary and match statistics
- Women's, A-team, under-19, Associate Member, domestic and franchise cricket
- User accounts, favourites, notifications and push alerts
- News, videos, images and player profiles
- Native mobile apps
- Any data source other than CricAPI

## Further Notes

- **Open values to fix before the first Feature is specified:** the refresh interval, the Stale Score threshold, and how long a Recently Completed Match stays visible. The first two depend on the CricAPI plan's request quota.
- **To verify before the first Feature:** CricAPI's free-tier request quota and its terms of use for this product. Both were assumed, not checked, when CricAPI was chosen (ADR CLP-0001).
- A Scorecard view is the most likely next Feature after the POC. The glossary already defines Scorecard.
