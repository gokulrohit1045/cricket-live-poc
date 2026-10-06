# Cricket Live

Live scores for men's international cricket between the ICC's Full Members. This glossary is the language every Spec, Plan, Story and conversation about the product uses.

## Language

### Teams

**Full-Member Team**:
The men's senior national team of one of the 12 ICC Full Members: Afghanistan, Australia, Bangladesh, England, India, Ireland, New Zealand, Pakistan, South Africa, Sri Lanka, West Indies and Zimbabwe.
_Avoid_: big team, major team, top nation

**National Team**:
A team representing a country (or, for West Indies, a group of countries); the product covers only those that are a **Full-Member Team**.
_Avoid_: country (as a team name)

### Matches

**International Match**:
A men's Test, ODI or T20I in which both teams are a **Full-Member Team**; a match against an Associate Member's team is not one, even though the ICC recognises it as international.
_Avoid_: international league, game, fixture (for a match already underway)

**Match**:
Short for **International Match**; at any moment exactly one of a **Fixture**, a **Live Match** or a **Completed Match**.

**Fixture**:
An **International Match** whose toss has not yet happened.
_Avoid_: upcoming game, schedule entry

**Live Match**:
An **International Match** between the toss and its **Result**, including every break in play — drinks, lunch, tea, rain or bad-light delays, the interval between innings, and overnight between the days of a Test.
_Avoid_: ongoing match, in-progress game

**Completed Match**:
A **Match** that has a **Result**.
_Avoid_: finished game, past match

**Recently Completed Match**:
A **Completed Match** still shown with its **Result** for a short window after it ends; once the window closes it is no longer shown. Not a **Live Match**.
_Avoid_: just finished, recent result

**Format**:
The kind of **International Match**: Test (up to five days, two **Innings** per side), ODI (one **Innings** per side, up to 50 overs each) or T20I (one **Innings** per side, up to 20 overs each).
_Avoid_: match type, version

**Result**:
How a **Match** ended — a win (with its margin in runs or wickets), a tie, a draw (Tests only), no result, or abandoned.
_Avoid_: outcome, final score

### Scoring

**Innings**:
One team's turn to bat in a **Match**.

**Score**:
The one-line summary of a **Match**'s current state — every **Innings** total so far, runs/wickets and overs in the current **Innings**, and the **Target**, lead or deficit where one applies; a Test's **Score** also gives the day (e.g. `IND 245/6 (42.3 ov) — target 290`, `ENG 325 & 140/3 (45 ov) v IND 280 — ENG lead by 185 · Day 3`).
_Avoid_: scoreline, live score (as a separate term)

**Target**:
The runs the side batting second must reach to win an ODI or T20I — always the current figure, including any revision under the DLS method, which the **Score** marks (e.g. `target 245 (DLS, 40 ov)`).
_Avoid_: revised target (as a separate term), chase total

**Scorecard**:
The full detail of each **Innings** of a **Match** — each batter's runs and dismissal, each bowler's figures, extras and the fall of wickets.
_Avoid_: full score, match details

### Data freshness

**Score Feed**:
The licensed source of **Fixtures**, **Scores**, **Scorecards** and **Results**; no other source of match data exists in the product.
_Avoid_: API, provider (in domain conversation), data source

**Last Updated**:
The time the **Score Feed** last provided data for a **Match**.
_Avoid_: refreshed at, timestamp

**Stale Score**:
A **Score** whose **Last Updated** time is too old for it to be trusted as the current state of a **Live Match**.
_Avoid_: old score, cached score

**Feed Delay**:
A period in which the **Score Feed** is unavailable or has not updated a **Live Match** for longer than expected.
_Avoid_: outage, lag

## Relationships

- A **Match** is played between exactly two **Full-Member Teams** in exactly one **Format**
- A **Match** moves **Fixture** → **Live Match** (at the toss) → **Completed Match** (at its **Result**), never backwards
- A **Match** has zero or more **Innings**: at most two in an ODI or T20I, at most four in a Test
- A **Completed Match** is a **Recently Completed Match** only during the window right after its **Result**
- Every **Score** carries a **Last Updated** time; a **Score** past its freshness limit is a **Stale Score**
- A **Scorecard** belongs to exactly one **Match** and is summarised by its **Score**

## Example dialogue

> **Dev:** "Rain stopped play in the India–England Test an hour ago. Is that still a **Live Match**?"
> **Domain expert:** "Yes — a **Live Match** runs from the toss to the **Result**, breaks and all. If the **Score Feed** hasn't sent anything for longer than expected, that's a **Feed Delay**, and we show it rather than a **Stale Score**."
> **Dev:** "And Ireland against Scotland tomorrow — is that a **Fixture**?"
> **Domain expert:** "No. Scotland isn't a **Full-Member Team**, so it isn't an **International Match** in this product at all."

## Flagged ambiguities

- "International league" was used for the product's scope — resolved: there is no league; the scope is **International Matches** between **Full-Member Teams**.
- "Big national team" — resolved: means a **Full-Member Team**, all 12 of them, with no shortlist.
- "Live score" was used for both the one-line summary and the full detail — resolved: the summary is the **Score**, the detail is the **Scorecard**.
