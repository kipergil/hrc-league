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
  players:["Tony Martin","Steve Hooker","Jake Skull"] },
```

- **Names are stored in full**, Title Case: `Derek Balding`, `Jackie Turner`.
  Sheets arrive in all caps (HRC B, HRC D), as initials (HRC C) or as a grid
  (HRC A) — convert all of them to this form. Surnames come from the club's own
  page (§2).
- **This is what keeps the roster unambiguous.** It is a single global list, so
  bare first names would merge two people into one filter entry. The club really
  does have two Johns — Barnes (HRC C) and Chamberlain (HRC A) — and two Nashes,
  and two Skulls.
- **Spell each player identically everywhere.** The roster is built by collecting
  distinct names across all fixtures, so a typo silently creates a new player.
- **Omit `players` entirely** when no line-up is published. Don't use `[]` — an
  empty array reads as "a line-up was published naming nobody", and the fixture
  would then be hidden from every player filter.

### Cup rounds

Cup rounds are one shared row applying to all four HRC teams, so line-ups are
keyed by team:

Several teams can name a line-up for the same round. The first Divisional round
already carries two:

```js
{ date:"2026-09-14", cupType:"Divisional",
  lineups:{ "HRC C":["Jackie Turner","John Barnes","Dave Cocks"],
            "HRC D":["Tony Martin","Jake Skull","Manuel"] } },
```

With one team in view each list is labelled "Playing"; across all teams it's
labelled with the team it belongs to, so several can sit on the one row without
ambiguity.

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
| HRC A | Andrew Nash, Chris Wade, Derek Balding, Kai Drake, Neil Skull, Paul Jones, Sandy Nash | team availability grid, first half only |
| HRC B | Anuj Patel, Gideon Alao, Mustafa Kipergil, Rai Liiv, Sunil Trakru | club sheet, first half only |
| HRC C | Dave Cocks, Dudu Souleiman, Faith Frankel, Jackie Turner, John Barnes, Mike Roberts | captain's Division One sheet, first half only |
| HRC D | Cathy Parsons, Jake Skull, Jo Swain, **Manuel**, Steve Hooker, Tony Martin | HL 26-27 Version 1, first half only |

### Surnames

Surnames come from the club's own page,
[`Clubz.asp?Club=HRC`](http://hertsttl.org.uk/Clubz.asp?Club=HRC), which lists
every registered player by team. Two things to know when reading it:

- **Registration is not selection.** The page groups players by the team they
  registered with, and its own note says players "play up (or down)". Gideon Alao
  and Mustafa Kipergil are registered HRC C but appear in HRC B's line-ups; Faith
  Frankel is registered HRC D but plays HRC C; Jake Skull is registered HRC C but
  plays HRC D. Take the surname from the page, the team from the line-up sheet.
- **It is not exhaustive.** Kai Drake's surname came from HRC A's own grid, and
  Tony Martin's from the club direct. **Manuel** is on no source at all and is
  still stored as a bare first name — add his surname when someone can supply
  it. Nothing collides with him, so he filters correctly meanwhile.
- **One name disagrees between sources.** The page registers him as *Andy* Nash;
  HRC A's own grid calls him *Andrew* Nash. The grid wins, as the team's own
  document, but it's worth confirming which he prefers.

The page also shows each team's contact and confirms all four HRC teams play
home matches on Wednesday.

### HRC A: a grid, not a list

HRC A's allocations arrive as a spreadsheet grid — fixtures down the side,
players across the top, a `1` where someone plays — rather than three names per
row. Two things follow:

- **It carries its own checksum.** A "Matches allocated" footer totals each
  player's column. Reconcile every column against it *and* check each row sums
  to 3; a reading that satisfies both is almost certainly right, and one column
  misread breaks at least one of them.
- **Red blocks mean unavailable**, not selected. They span the dates a player is
  away, and no tick ever appears inside one — useful as a third check.

**John Chamberlain is on the grid with 0 matches.** He therefore appears nowhere
in the data and won't be offered in the filter — correct, not a missed column.
He is one reason names are stored in full: bare first names would have collided
with HRC C's John Barnes the moment Chamberlain was picked.

### HRC C initials

The captain's sheet allocates players by initials. Keep this mapping to decode
the next one:

| Initials | Full name | Stored as |
|---|---|---|
| DS | Dudu Souleiman *(captain)* | `Dudu Souleiman` |
| JB | John Barnes | `John Barnes` |
| JT | Jackie Turner | `Jackie Turner` |
| DC | Dave Cocks | `Dave Cocks` |
| MR | Mike Roberts | `Mike Roberts` |
| FF | Faith Frankel | `Faith Frankel` |

`DS`/`DC` and `JB`/`JT` differ only in the second letter, so read them
carefully. Faith's surname is illegible on the handwritten sheet — the club page
settles it as **Frankel**, not Frances.

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

Club pages live at `hertsttl.org.uk/Clubz.asp?Club=<name>` and carry the venue
addresses and start times in §5. There is also a
[Google Maps list of every venue](https://maps.app.goo.gl/2CZiqXpNb9RbLioFA?g_st=i)
("Herts TTL – Venues"), kept by Mustafa and linked from the page footer — handy
for a season overview, though the per-fixture pins are built from the addresses
below rather than from that list.

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

This formula reproduces **all 30** fixtures the club sheets and the league grids
both cover, to the day:

| Sheet | Fixtures | Notably |
|---|---|---|
| HRC B, first half | 7 | Friday away at Ellenborough A |
| HRC D, first half | 8 | Thursday opener at St. Andrews B, Tuesday at Cheshunt D |
| HRC C, first half | 8 | Monday at Grundy Park B, Thursday at Ellenborough B |
| HRC A, first half | 7 | Tuesday at Cheshunt A |

Four sheets, in four different formats, written by different people, none of
whom worked from this formula — and every date agrees. The non-Wednesday ones carry the most weight:
each is an away fixture landing on that opponent's own home night, which is the
formula's least obvious consequence.

The sheets' one remaining row, HRC D's Water Lane C fixture, had no league
counterpart at all and turned out to be an error — see §7.

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

### Venues and start times

From each club's own page at `hertsttl.org.uk/Clubz.asp?Club=<name>`. The venue
is a property of the **club**, so a fixture is played at whichever club is at
home — the calendar export derives it the same way it derives the date.

| Club | Venue | Start |
|---|---|---|
| Cheshunt | Thundridge Village Hall, Ermine Street, Thundridge, Herts. SG12 0SY | 7.30pm |
| Ellenborough | Ellenborough Club, Craddock Road, Enfield. EN1 3SP | 7.30pm |
| Furneux Pelham | Furneux Pelham Village Hall, Barleycroft End, Furneux Pelham. SG9 0LL | **7pm** |
| Grundy Park | Laura Trott Leisure Centre, 44 Windmill Lane, Cheshunt, Herts. EN8 9AJ | 7.30pm |
| **HRC** | Bushby Hall, 8 Wharf Road, Wormley, Herts. EN10 6HX | 7.30pm |
| Kidston | Kidston Institute, 7 Northaw Rd West, Northaw, Potters Bar, Herts. EN6 4NW | 7.30pm |
| PramaStars | Stanstead Abbotts Parish Hall, Roydon Road, Stanstead Abbotts. SG12 8EZ | 7.30pm |
| St. Andrews | St. Andrews Centre, St Andrew St, Hertford. SG14 1HZ | 7.30pm |
| Stanstead Abbotts | Stanstead Abbotts Parish Hall, Roydon Road, Stanstead Abbotts. SG12 8EZ | 7.30pm |
| Water Lane | Herts & Essex Sports Centre, Beldams Lane, Bishops Stortford. CM23 5LH | **7pm** |

7.30pm is the league norm and isn't stated on most club pages. Only two ask for
something else, both in their own words:

- **Water Lane** — "We have the hall from 7pm til 10pm on Wednesdays so, please,
  all Water Lane home matches to start as close to 7pm as possible."
- **Furneux Pelham** — "Please can all our home matches start at 7pm."

Water Lane's page adds that on **Friday** nights they only have the hall until
9pm. No HRC fixture is affected — every Water Lane team hosts on a Wednesday —
but it would matter if a fixture were rearranged.

PramaStars and Stanstead Abbotts share a venue, so a fixture against either is
at the same hall.

**The St. Andrews Centre needs a different search term.** It sits behind
St Andrew's Church, as the club's page notes, and Google Maps doesn't find the
hall by its own name or postcode — so its venue entry carries a `maps` override:

```js
"St. Andrews": { place:"St. Andrews Centre", address:"St Andrew St, Hertford. SG14 1HZ",
                 maps:"Hertford St Andrew Church" },
```

`maps` changes only what gets searched. The address still shows on the page and
in the calendar entry's `LOCATION`, so nobody is told to go to a church. Add the
same field to any other venue whose address doesn't land on the right pin.

**Match length isn't published anywhere.** The calendar export blocks 2½ hours,
which puts a 7.30pm start at 10pm; Water Lane's hall booking is the only
evidence of a match's length. Treat it as an estimate, not a fact.

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

**Minor typos on the HRC D sheet, already handled:** "Fernaux Pelham" is the
league's *Furneux Pelham*; the 21/09 row's date reads "24/09/25" for 2026.

**The HRC C sheet was clean** — 8/8 rows matched the grid on date, weekday,
opponent and home/away, with no extra or missing fixtures in either direction.
Worth recording: the checklist isn't there because sheets are usually wrong, but
because one was, and nothing else would have caught it.

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

- **Second halves for all four teams.** HRC C's sheet notes that 2027 fixtures
  are "to be decided nearer the time", so expect those sheets late.
- **Cup rounds** — only two are assigned: 14 Sep (HRC C and HRC D) and 12 Oct
  (HRC D). The other seven are unassigned.
