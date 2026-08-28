# Hertford League 2026/27 — data reference

Everything needed to keep `index.html` accurate: where the fixture data comes
from, how match dates are derived, which night each club plays, and the exact
steps for adding a new line-up sheet.

The calendar is a single self-contained file. All data lives in two arrays near
the top of the `<script>` block in `index.html` — there is no build step and no
external data fetch at runtime.

---

## 1. Adding player line-ups

This is the common job: the club issues a fixture sheet with player allocations
and they need to go into the calendar.

### League matches

Find the fixture in the `matches` array and add a `players` key. Nothing else
changes — the date and opponent are already correct.

```js
{ team:"HRC D", date:"2026-09-30", ha:"Home", opponent:"Furneux Pelham",
  players:["Tony","Steve","Jake"] },
```

- **Names are Title Case** (`Tony`, not `TONY`) — club sheets are usually all
  caps, so convert them. The filter dropdown lists names exactly as written, and
  `"Tony"` and `"TONY"` would appear as two different players.
- **Spell each player identically everywhere.** The roster is built by collecting
  distinct names across all fixtures, so a typo silently creates a new player.
- **Omit `players` entirely** when no line-up is published. Don't use `[]` — an
  empty array reads as "a line-up was published naming nobody", and the fixture
  would then be hidden from every player filter.

### Cup rounds

Cup rounds are one shared row applying to all four HRC teams, so line-ups are
keyed by team:

```js
{ date:"2026-09-14", cupType:"Divisional",
  lineups:{ "HRC D":["Tony","Jake","Manuel"] } },
```

Several teams can name a line-up for the same round:

```js
lineups:{ "HRC B":["Anuj","Rai"], "HRC D":["Tony","Jake","Manuel"] }
```

Omit `lineups` when nobody is assigned — the round then shows "Applies to all
HRC teams" and stays visible to everyone.

### What the filters then do

| Fixture state | With a player filter on |
|---|---|
| Names the player | Shown, highlighted, their chip picked out |
| Names a line-up without them | Hidden — they aren't playing |
| No line-up published | **Shown**, marked "Line-up not announced yet" |
| Cup round, no line-up for their team | Shown, "Applies to all HRC teams" |

The third row is deliberate: an unallocated fixture isn't a fixture without that
player, it's one nobody has been picked for yet.

---

## 2. Current rosters

Taken from the line-ups presently in the data.

| Team | Players | Source |
|---|---|---|
| HRC A | *none yet* | — |
| HRC B | Anuj, Gideon, Mustafa, Rai, Sunil | club sheet, first half only |
| HRC C | Dave, Dudu, Faith, Jackie, John, Mike | captain's Division One sheet, first half only |
| HRC D | Cathy, Jake, Jo, Manuel, Steve, Tony | HL 26-27 Version 1, first half only |

### HRC C initials

The captain's sheet allocates players by initials. Keep this mapping to decode
the next one:

| Initials | Full name | Stored as |
|---|---|---|
| DS | Dudu Souleiman *(captain)* | `Dudu` |
| JB | John Barnes | `John` |
| JT | Jackie Turner | `Jackie` |
| DC | Dave Cocks | `Dave` |
| MR | Mike Roberts | `Mike` |
| FF | Faith Frances | `Faith` |

`DS`/`DC` and `JB`/`JT` differ only in the second letter, so read them
carefully. Faith's surname is hard to make out on the sheet — it doesn't matter
while first names are stored, but check it before switching to full names.

A team with no line-ups anywhere hides the player dropdown entirely when it's
selected. That's expected, not a bug.

---

## 3. Sources

| Division | Grid page | Date logic |
|---|---|---|
| Premier | [Calendar0.htm](http://hertsttl.org.uk/Calendar0.htm) | [CalendarJ0.js](http://hertsttl.org.uk/CalendarJ0.js) |
| Division One | [Calendar1.htm](http://hertsttl.org.uk/Calendar1.htm) | [CalendarJ1.js](http://hertsttl.org.uk/CalendarJ1.js) |
| Division Two | [Calendar2.htm](http://hertsttl.org.uk/Calendar2.htm) | [CalendarJ2.js](http://hertsttl.org.uk/CalendarJ2.js) |

HRC A and HRC B are in the Premier Division, HRC C in Division One, HRC D in
Division Two.

Each grid is a table of teams (rows) against weeks (columns). Every fixture cell
carries its own handler, which is the reliable way to read the grid — column
counting is not:

```html
ONMOUSEOVER="Match('B','I','h','2','0','0')"
              │    │    │    │
              │    │    │    └─ week number
              │    │    └────── 'h' home, 'a' away (for the row's team)
              │    └─────────── opponent index
              └──────────────── the row's team index
```

Indices map to teams through `aIndex` (`"A"`…`"J"`) and `aTeam` in that
division's JS. `Cup(...)` and `Break(...)` cells mark cup weeks and byes.

---

## 4. How match dates are derived

**The date shown in each column header is the week commencing (a Monday), not
the match date.** The real date appears only in the hover tooltip, and it isn't
stored anywhere — the JS computes it:

```
match date = season start + (week − 1) × 7 days + hosting club's home night
```

- **Season start:** Monday **14 September 2026** (`dStartOfSeason`, identical in
  all three divisions).
- **Home night:** `aHomeNight`, an offset where **0 = Monday**, so 2 = Wednesday.
- **Hosting club:** the home team. For an away fixture this is the *opponent*,
  which is why away dates follow the opponent's night, not HRC's.

### Worked example

HRC D away to St. Andrews B, week 2. St. Andrews B play Thursdays (offset 3):

```
14 Sep 2026 + (2 − 1) × 7 + 3  =  24 Sep 2026, a Thursday
```

Which matches the club sheet exactly.

### Confidence

This formula reproduces **all 15** fixtures the two club sheets and the league
grids both cover — HRC B's first half (7) and HRC D's (8) — to the day,
including the awkward ones: the Friday at Ellenborough A, the Tuesdays at
Cheshunt. Two independent sources agreeing on every date either can confirm is
why these dates are trusted. The sheets' one remaining row, HRC D's Water Lane
C fixture, had no league counterpart at all and turned out to be an error — see
§7.

---

## 5. Clubs and home nights

The night is a property of the **club**, so it decides the date whenever that
club is at home — including HRC's away fixtures.

### Premier Division

| Club | Home night |
|---|---|
| Cheshunt A | Tuesday |
| Ellenborough A | Friday |
| Grundy Park A | Monday |
| **HRC A** | **Wednesday** |
| **HRC B** | **Wednesday** |
| Kidston | Wednesday |
| Water Lane A | Wednesday |
| Water Lane B | Wednesday |

### Division One

| Club | Home night |
|---|---|
| Cheshunt B | Tuesday |
| Cheshunt C | Tuesday |
| Ellenborough B | Thursday |
| Grundy Park B | Monday |
| Grundy Park C | Monday |
| **HRC C** | **Wednesday** |
| PramaStars A | Friday |
| St. Andrews A | Thursday |
| Water Lane C | Wednesday |

### Division Two

| Club | Home night |
|---|---|
| Cheshunt D | Tuesday |
| Furneux Pelham | Wednesday |
| Grundy Park D | Monday |
| **HRC D** | **Wednesday** |
| PramaStars B | Friday |
| St. Andrews B | Thursday |
| Stanstead Abbotts | Thursday |
| Water Lane D | Wednesday |
| Water Lane E | Wednesday |

All four HRC teams play home matches on **Wednesday**. A fixture on any other
day is an away match, and the day tells you which club is hosting.

---

## 6. Season week calendar

Week 1 begins Monday 14 September 2026; every week runs Monday to Sunday.

| Wk | W/c | What's on |
|---|---|---|
| 1 | 14 Sep 2026 | Divisional Cup |
| 2 | 21 Sep 2026 | League |
| 3 | 28 Sep 2026 | League |
| 4 | 05 Oct 2026 | League |
| 5 | 12 Oct 2026 | Divisional Cup |
| 6 | 19 Oct 2026 | League |
| 7 | 26 Oct 2026 | League |
| 8 | 02 Nov 2026 | League |
| 9 | 09 Nov 2026 | Divisional Cup |
| 10 | 16 Nov 2026 | League |
| 11 | 23 Nov 2026 | Free week — no fixtures |
| 12 | 30 Nov 2026 | League |
| 13 | 07 Dec 2026 | League |
| 14 | 14 Dec 2026 | Cup Finals |
| 15–16 | 21, 28 Dec 2026 | Christmas break |
| 17 | 04 Jan 2027 | Handicap Cup |
| 18 | 11 Jan 2027 | League |
| 19 | 18 Jan 2027 | League |
| 20 | 25 Jan 2027 | League |
| 21 | 01 Feb 2027 | Handicap Cup |
| 22 | 08 Feb 2027 | League |
| 23 | 15 Feb 2027 | League |
| 24 | 22 Feb 2027 | Handicap Cup |
| 25 | 01 Mar 2027 | League |
| 26 | 08 Mar 2027 | League |
| 27 | 15 Mar 2027 | League |
| 28–29 | 22, 29 Mar 2027 | Break |
| 30 | 05 Apr 2027 | Handicap Cup |
| 31 | 12 Apr 2027 | League |
| 32 | 19 Apr 2027 | Cup Finals |

Cup weeks are league-wide and stored once in `cupRounds`, dated on the Monday —
the grids give cup rounds no specific night.

### HRC bye weeks

On a league week, a team with an odd fixture count sits out. Useful when a
fixture seems to be missing.

| Team | Bye weeks |
|---|---|
| HRC A | 3, 11, 12, 15, 16, 19, 27, 28, 29 |
| HRC B | 8, 10, 11, 15, 16, 25, 26, 28, 29 |
| HRC C | 11, 12, 15, 16, 27, 28, 29 |
| HRC D | 7, 11, 15, 16, 23, 28, 29 |

---

## 7. Sheet errors found so far

**HRC D vs Water Lane C, 28 Oct 2026 — a spurious row. Resolved: not in the
calendar.** The HL 26-27 Version 1 sheet listed this away fixture with a
line-up of Jake, Cathy and Jo. Three checks put it beyond doubt:

- Week 7 in HRC D's row is a `Break` cell reading "No Match" — their bye week.
- Water Lane C is a **Division One** club. It's on HRC C's fixture list, not
  HRC D's.
- It isn't a mis-spelling of Water Lane D or E either. HRC D's two Water Lane
  away fixtures are already on the sheet, on their correct dates and with
  different line-ups — E on 07/10, D on 18/11 — so there is no missing fixture
  for this row to be. The sheet appears to have filled in a bye week.

The line-up went with it. All six players still appear elsewhere, so the roster
is unaffected.

This is the case the §8 checklist exists for: it was caught only by
cross-checking each sheet row against the league grid, and it briefly put an
extra fixture in the calendar.

**Minor sheet typos, already handled:** "Fernaux Pelham" is the league's
*Furneux Pelham*; the 21/09 row's date reads "24/09/25" for 2026.

**Nothing outstanding.** Every fixture in the calendar now matches the league
source exactly.

---

## 8. Applying a new sheet — checklist

1. **Dates:** don't take them from the sheet's week-commencing column. Use the
   Date column, or derive from §4. The two agree; when they don't, the league
   grid wins.
2. **Cross-check every fixture against the grid** before adding it. That check
   is what caught the Water Lane C row.
3. **Convert names to Title Case** and match existing spellings exactly.
4. **Attach line-ups** per §1 — `players` on league matches, `lineups` on cup
   rounds. Omit rather than empty.
5. **Verify counts:** players × 3 slots should equal fixtures × 3. The per-player
   appearance count is a quick way to catch a misread row.
6. **Update the footer count** in `index.html` if the fixture total changes.
7. **Check in a browser:** pick the team, then each player, and confirm the
   summary counts match the sheet.

### Outstanding

Dates are complete: all four teams use real match dates, and every fixture
matches the league source.

Line-ups still to come:

- **HRC A** — none at all, either half.
- **HRC B, C, D** — second half only. HRC C's sheet notes that 2027 fixtures are
  "to be decided nearer the time", so expect those sheets late.
- **Cup rounds** — only the first Divisional round has line-ups (HRC C and
  HRC D). The rest are unassigned.
