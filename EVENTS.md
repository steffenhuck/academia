# Writing events for The Discipline

Events are pure data. Add one by pasting an object into the `EVENTS` array in
`index.html` — no other code needs to change. The engine handles selection,
gating, repeat-suppression, effects and display.

---

## The shape

```js
{id:"A8", ti:"The Dean's Note", b:[2,4], ex:{base:1, highP:1.4}, ga:{N:40}, once:1, tr:"pres",
 sc:"Scene text. HTML is allowed — <em>italics</em> and <br><br> for a break.",
 ch:[
  {t:"The button label.", fx:{K:6, N:3}, say:"What happens. One or two sentences."},
  {t:"The other one.", roll:{stat:"N", pivot:50,
    win: {fx:{N:8},  say:"When it works."},
    lose:{fx:{E:-6}, say:"When it doesn't."}}}
 ]}
```

| field | meaning |
|---|---|
| `id` | unique string. Prefix by stage: `P` PhD, `D` postdoc, `A` assistant, `S` senior/admin, `N` senior/research, `G` any |
| `ti` | title, shown large in the display serif |
| `b` | `[lowest rank, highest rank]` the event can fire at — see ranks below |
| `ex` | exposure weights: which situations make this more likely |
| `ga` | gate: the event can't fire unless these are met |
| `once` | `1` if it should only ever happen once in a career |
| `tr` | `"nobel"` or `"pres"` — weights it toward that vow, against the other |
| `sc` | the scene |
| `ch` | the choices, 1–4 of them — **or** `auto` instead, see below |
| `auto` | `{fx, say, good:1}` or `{fx, say, bad:1}` — an event with no choice. The thing simply happens; the card shows the scene and the outcome together. Use for scoops, policy fights, asbestos, a former student's note |

**Ranks:** `0` PhD student · `1` postdoc · `2` assistant professor · `3` associate
· `4` full professor. Administrative office is tracked separately, so a dean is
still rank 3 or 4.

---

## Choices

| field | meaning |
|---|---|
| `t` | the button label |
| `why` | optional grey subtitle under the label |
| `fx` | what it does (see below) |
| `say` | the outcome text, shown after clicking |
| `roll` | instead of `fx`: a gamble, with `win` and `lose` branches |
| `gate` | locks the option behind a requirement, showing why |

**`fx` keys:**

- `R` `K` `N` `P` — research, know-how, network, public profile (0–100)
- `S` — salary, in $k, unscaled
- `E` `B` — ego, burnout risk (0–100), unscaled
- `wip` — pages toward the next paper (100 = a submission)
- `mom` — momentum, the hidden Matthew-effect multiplier
- `cites:60` — a jolt to one existing paper's citation count (a policy fight, a
  textbook box, a student who cites you in everything). Does nothing if there
  are no papers yet
- `flag:"NAME"` — sets a permanent flag; add it to `EPITHETS` to have it appear
  in the ending's *"The field remembers…"* line
- `foe:1` — makes an enemy, with a generated name, grudge and what they poison
- `foe:{why:"you rejected their paper", poisons:"committee"}` — a specific
  enemy. `why` is what they remember; `poisons` is where it bites:
  `"referee"` lowers the odds of a good journal on every submission,
  `"vote"` and `"committee"` are named no-votes at tenure, `"committee"` also
  hurts the October roll, `"search"` hurts the presidential search
- `mod:"name"` — a one-shot engine modifier. Two do something immediately:
  `"resignOffice"` gives up any administrative post; `"toCranmoor"` leaves a
  postdoc for a tenure-track line at Cranmoor A&M with a six-year clock
- `tax:8` — the choice costs one allocation point per term for that many terms
  (editorial boards, co-editorships). Taxes stack; the budget never drops below 3
- `appoint:"PELHAM"` — makes the player president of that institution outside
  the search: the exit-upward offer, the Harwich committee choosing you
- `clearFoe:"search"` — removes one enemy of that kind (serving on the Harwich
  search and making sure, on the merits)
- `mod:"nobelDead"` — closes the Nobel for good; presidents are not called
- `ending:"EXIT"` — ends the career immediately

**`roll`:** `stat` is one of `R` `K` `N` `P` `E`, or `"coin"` for a pure 50/50
that no stat can move — use it for the things that really are luck: whether the
star stays, whether the fund was fraud, whether the brilliant student's draft
was brilliant. Otherwise `pivot` is the value at which it's a coin flip
(default 50). Ego shifts the odds in your favour, scales the
damage when you lose, and can spawn an enemy on a miss — so a gamble is a
different proposition for an arrogant player.

---

## Magnitudes

Positive `R`/`K`/`N`/`P` gains are scaled by how full the counter already is, so
a raw `+8` moves a player at 50 by about 4, and a player at 85 by about 1.5.
Write the raw number and let the engine handle the rest. **Losses are not
scaled** — it is easier to wreck a reputation than to build one.

| size | raw value | use for |
|---|---|---|
| small | 3–5 | an ordinary term's worth of consequence |
| medium | 6–9 | a real decision with a real cost |
| large | 10–14 | a set-piece, usually `once:1` |

Salary moves in real money: `S:6` is a decent raise, `S:14` a successful
counteroffer. Burnout: `B:5` is a heavy term, `B:14` a brutal one.

---

## Exposure weights (`ex`)

Each key is multiplied by a value between 0 and 1 describing the player's
current situation, then summed, then exponentiated. `base:1` means "can happen
any time"; the others make an event find the player it belongs to.

| key | high when |
|---|---|
| `base` | always — the baseline weight |
| `highR` `highK` `highP` | research / know-how / profile is high |
| `highB` | burnout risk is high |
| `highMom` `lowMom` | momentum is running hot / cold |
| `lowS` | salary is low |
| `tdebt` | teaching has been neglected |
| `spec` | the player once used the defensible specification |
| `office` | holds any administrative post |
| `highE` | ego is high — for the temptations ego makes likelier |

A weight of `1` is a normal pull, `2` a strong one. Every draw also has a
"quiet term" weight in the pot, so nothing is guaranteed.

---

## Gates (`ga`, and per-choice `gate`)

`{R:45}` `{K:50}` `{N:40}` `{P:60}` `{E:70}` `{flag:"SPEC"}` — any combination.
Also: `{office:1}` (at least department chair; `2` dean, `3` provost, `4`
president), `{tenured:1}`, `{flags:["A","B"]}` — all of several flags — and
`{since:["FLAG",6]}` — the flag was set at least six terms ago. `since` is how a decision comes back years later: the breach
you handled quietly becomes the blackmail in `C6`; the postdoc who said nothing
in `P9` gets `G13` three years on.
On an event, an unmet gate means it can't fire. On a choice, the button is
shown disabled with a reason, which is often better than hiding it: seeing what
you can't do yet is part of the picture.

---

## Flags currently in play

Every flag set by an event is listed in `EPITHETS` in the source, with the line
it produces at the end. The ones other events currently gate on:

| flag | set by | gates |
|---|---|---|
| `SPEC` | P6 the defensible specification | N6 the replication |
| `LEFT_ERROR` | P7 leaving the lemma wrong | G12 the graduate student, 8+ terms later |
| `REPLACED_SILENT` | P9 going home quietly | G13 the paper you weren't on, 6+ terms later |
| `COEDITOR` | N7 co-editing the Review | E1 the laureate's paper, E2 the glowing report |
| `QUIET_BREACH` | C5 fixing the breach yourself | C6 the envelope, 4+ terms later |
| `FIRST_STUDENT` | A10 taking the first student | S7 the student at Harwich, 16+ terms later |
| `UNCITED_SOURCE` / `CITED_OLD` | G14 page 411 | G15 the provenance paper, or G16 Budapest |
| `TOOK_IDEA` / `GAVE_IDEA` | N11 the student's idea | N12 / N13 the acceptance speech, 14+ terms later |
| `CHOSE_SMART` / `CHOSE_DILIGENT` | A15 one place | A16 variance / A17 the floor |
| `LISBON` | G17 | G18 the second household |
| `AFFAIR_JUNIOR` | G19 both adults | G20 through the proper channel — pulled by office and profile |
| `ASKED_SELF_CITE` / `KILLED_RIVAL` | R1 it doesn't cite you | E3 / E4, once you co-edit and must sign |
| `BET_STAR` / `BET_FUND` / `BET_GULF` | Q3 / Q5 / Q7 | Q4 / Q6 / Q8, eight terms later, on a coin |
| `HR_ENABLING` | Q14 head of HR | Q15 enabled, six terms later |
| `BET_GENIUS` → `GENIUS_LISTED` | Q16 the potential laureate | Q17 the call next door (coin), then Q18 the star's past |
| `WARNED_HIM` | G24 the spreadsheet | G25 heads up |
| `WATCHED` | G26 the laureate's hand | G27 the essay, eight terms later |
| `PROTECTED_STAR` | C7 the most cited member | C8 what the chair knew |

New flags are free — set one with `flag:"WHATEVER"`, gate a later event on it
with `ga:{flag:"WHATEVER"}`, and add a line to `EPITHETS` if it deserves to be
remembered at the end. Long-fuse consequences are the cheapest good thing in
the system.

---

## Publication is an event

When a paper comes out, the term's card *is* the publication — title, journal,
tier, and a line that depends on the tier. It takes the slot an event would
have taken. Nothing to write for this; it happens in the engine.

## Endings that events can now reach

A Nobel-vow player who accepts a presidency (N14, or the Harwich committee in
G21) gets **The exit upward** — the office, the salary, and a phone that does
not ring for presidents. The vow is unmet, so the card closes on salary.

## What exists (136 events)

Events *available* at each rank, counting the ones whose band spans it:

| rank | available | still thin on |
|---|---|---|
| PhD (0) | 13 | the market itself, the cohort as a group, money |
| postdoc (1) | 31 | the second market, being adjacent to power |
| assistant (2) | 55 | the clock's last two years |
| associate/full (3–4) | 95–99 | decline, the long plateau, being overtaken |
| chair / dean / provost | 12 | fundraising, the provost's office specifically |
| president | 14 | the board as an ongoing relationship, the second term |

The engine is built for roughly 200. Nothing about adding them is structural —
it is writing.

---

## Notes

- `alt:[...]` on an event gives alternate scene text for repeat firings, so a
  recurring event doesn't read identically the third time. Worth it for
  anything without `once:1` that will fire often.
- Teaching is deliberately close to worthless. Events that make it matter
  should do so by *punishing neglect*, not by rewarding investment.
- The best events give the player a choice where both options are defensible
  and one of them quietly costs something that won't be visible for twenty
  years.
