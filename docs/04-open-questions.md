# Open Questions

Decisions needed before or during Phase 0. The four marked **★** change the shape of the work and should be settled first.

---

## Backend and data

**★ Q1 — What access is there to the existing database?**
The site runs Classic ASP on IIS/Plesk, so the data is most likely SQL Server or Access. A direct export makes migration straightforward. If it isn't available, everything must be scraped from the public HTML — proven possible in this audit, but it loses anything the site doesn't display (email addresses, phone numbers, historic scorecards behind login) and adds roughly two weeks.
*Options:* full export available · read‑only access available · scrape only · unknown, needs investigation.

**Q2 — Who currently holds the hosting, DNS and domain, and can that be transferred?**
Needed for the HTTPS fix regardless of anything else.

**Q3 — How far back should the archives go?**
The Hall of Fame runs to 1970. Tables, averages and match results exist on the site for 2024‑25 and 2025‑26 only. Is there anything older in the database, in spreadsheets, or in a filing cabinet? Digitising more history is high‑value and low‑risk, but only if it exists.

---

## Hosting and operations

**★ Q4 — Managed or self‑hosted?**
Directus Cloud costs roughly £180–250/year and removes patching, backups and uptime from a volunteer. Self‑hosting on a small VPS is roughly £90–160/year but needs someone to look after it. Given the bus‑factor risk this project exists partly to solve, managed is recommended.
*Options:* Directus Cloud · self‑hosted VPS · keep the existing Plesk host · undecided.

**Q5 — Who runs the site after launch, and how much time do they have?**
This determines how much can safely be automated versus how much needs an interface. If the answer is "the same one volunteer", the scope should shrink and the automation should grow.

**Q6 — What is the annual budget?**
The plan assumes under £250/year. If there is less, self‑hosting and free tiers still work; if there is more, a professional accessibility audit is the best place to spend it.

---

## Scope and sequencing

**★ Q7 — Phase 1 scope: public site only, or public site plus captain result entry?**
The public read‑only site (Phase 2) can launch in about 9 weeks and fixes every critical and high finding. Result entry (Phase 3) adds about 4 weeks and is where the real workflow benefit sits — but it also carries the change‑management risk, because it asks captains to change a habit.
*Options:* public site first, entry later · both together · public site only for now.

**★ Q8 — How much visual change do you want?**
Older users value familiarity, and a radical redesign can cost more in confusion than it gains in polish. Three positions: (a) keep the current structure, colours and page names, fixing only legibility and mobile; (b) a clear modern redesign that keeps the same information architecture and page names; (c) a full rethink of both look and structure. Recommendation: **(b)**.

**Q9 — Should the old site stay online?**
Recommendation: freeze it read‑only at `legacy.hertsttl.org.uk` for one season as a safety net, with all old URLs 301‑redirected to the new site.

**Q10 — When should cutover happen?**
Recommendation: pre‑season, late August. The alternative safe window is the Christmas break. Never mid‑season or near a cup final.

---

## People, privacy and access

**Q11 — Who should be able to see player contact details?**
Today they are nominally behind a login, but that login is an email address with no password, so in practice they are public. Options: all logged‑in players; only captains and committee; only committee; or per‑person consent (recommended — the data model supports it).

**Q12 — Do juniors play in the league, and how many?**
There is a Junior Singles event in the Closed Championships. If under‑18s are registered, parental consent records and stricter contact‑data handling are required, and the plan needs a small addition.

**Q13 — How long should personal data be kept after a player leaves?**
The plan assumes three seasons for contact details, with names retained indefinitely in results and honours as league records. Needs committee ratification for the privacy notice.

**Q14 — Should players be able to opt out of appearing publicly?**
Results and averages are arguably league records, but a player may reasonably not want their name indexed by Google. Worth a stated position.

---

## Features

**Q15 — Should tournament entry fees be payable online?**
Currently "a nominal entry fee… on the night". Online payment adds Stripe, reconciliation and refunds — real complexity for a volunteer committee. Recommendation: keep it cash on the night for now.

**Q16 — Should the site email players directly, and about what?**
The plan proposes: result confirmation requests to captains, outstanding‑card chasers, optional fixture reminders, and the newsletter. Each needs a consent position and an unsubscribe route.

**Q17 — Is a "find a club near me" map wanted?**
The league covers Bishop's Stortford to Enfield — a wide area, and a genuine recruitment aid. It needs venue coordinates, which the data model already provides for.

**Q18 — Should player profile pages exist publicly?**
Career records across seasons are a nice feature and the data supports it, but some players will not want a permanent public record. Related to Q14.

**Q19 — Is a mobile app wanted, or is a Progressive Web App enough?**
Recommendation: PWA. It installs to a home screen, works offline for fixtures and draft scorecards, and needs no app store.

**Q20 — Should the committee minutes be public?**
They currently are, but the most recent published set is from 2017. Either commit to publishing them or remove the section — a nine‑year‑old "latest minutes" link reads worse than none.

---

## Content

**Q21 — Who will write the new content?**
The rebuild needs: an About/history page for the 90th anniversary, a "How the league works" explainer, a "New to the league?" join page, venue accessibility notes for each club, and a privacy notice. The league has the knowledge; it needs someone to write it down. Roughly two days of committee time.

**Q22 — Are there club logos, photographs or archive material available?**
The current site has almost no imagery. Photographs of venues in particular help first‑time visitors and prospective players far more than stock imagery would.

**Q23 — Is there a sponsorship model to preserve?**
The current site carries Thorntons, Bribar, KitUOut and a coaching academy. The data model has a `sponsors` collection with tiers and display windows — the question is whether these are paid placements with obligations attached.
