# Writing events for The Young Economist’s Game to Professional Success

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
- `submit:{...}` — **a real paper goes out now**, and its publication card
  arrives some terms later with a title and a journal. Options: `tier:"A+"`
  forces the tier; `bias:1.2` shifts the odds instead; `lag:[3,4]` terms to a
  decision; `q:8` quality; `f:"theory"` field; `jn:"the Handbook"` names the
  outlet; `cites:60` starts it with citations; `rounds:1` skips the R&R roll;
  `target:"A"` aims at a tier instead of rolling one — the paper can then be
  rejected and go down a tier, which is what the player's own "Where to send
  it" card does. If the scene named the paper with `{paper}`, that is the
  paper that goes out.
  An array submits several. **Use this, not prose, whenever a choice results
  in a paper** — the feedback should say it went out, and the card that comes
  later says it is in
- `mod:"accept"` / `"reject"` / `"resubmit"` / `"delay"` — act on the paper that
  has been under review longest: accepted next term; back in the drawer
  (returns 45 pages of draft); sent down a tier and out again; one more round.
  Events that use these must be gated `ga:{queue:1}` so a paper is under review
- `mom` — momentum, the hidden Matthew-effect multiplier
- `cites:60` — a jolt to one existing paper's citation count (a policy fight, a
  textbook box, a student who cites you in everything). Does nothing if there
  are no papers yet
- `flag:"NAME"` — sets a permanent flag; add it to `EPITHETS` to have it appear
  in the ending's *"The field remembers…"* line
- `foe:{why:"you rejected their paper", poisons:"committee"}` — an enemy.
  **Always write `why`**: it is shown as "X will remember that <why>", so it
  must be the thing that just happened in this event. (`foe:1` still works
  and draws from a neutral grudge list; it is only for the unscripted
  enemies the engine makes on its own.) `who:"{senior}"` names a cast
  member instead of a generated stranger. `poisons` is where it bites:
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
Also: `{office:1}` (at least department chair; `2` dean, `3` vice-president, `4`
president; the head of St Cuthbert's is called Provost), `{tenured:1}`, `{flags:["A","B"]}` — all of several flags — and
`{since:["FLAG",6]}` — the flag was set at least six terms ago;
`{anyflag:["A","B"]}` — at least one of several; `{noflag:"A"}` — the flag is
*not* set, for the other half of a pair like `Y9`/`Y9H`; `{tdebt:8}` — the
hidden teaching debt (one point a term without teaching, two off per teaching
point) has reached that level. `Y10` is how neglected teaching eventually
goes wrong in a room. `{nobelhope:1}` — a Nobel is still possible and
hoped for: not the presidency vow alone, not won, not dead, full professor,
fifty or over (`Z4`, the prank call).

`{queue:1}` — at least one paper under review; required by any event about the
review process.

**Gates on the record:** `{field:"macro"}` — has published at least one paper in
that field (fields: theory, micro, macro, metrics, finance, labour, development,
political, behavioural, history); `{h:20}` — h-index; `{cites:2000}`;
`{papers:3}`. This is how an event can be *about* what the player wrote: the
central bank adopts the macro model, the historian reviews the history paper,
the R package with the metrics estimator has a bug. `since` is how a decision comes back years later: the breach
you handled quietly becomes the blackmail in `C6`; the postdoc who said nothing
in `P9` gets `G13` three years on.
On an event, an unmet gate means it can't fire. On a choice, the button is
shown disabled with a reason, which is often better than hiding it: seeing what
you can't do yet is part of the picture.

---

## Storylines

A storyline is a list of event ids in order, with a gap in terms between steps:

```js
{id:"breach", steps:["C5","C6"], gap:[4,7]}
```

The first step is an ordinary event and is drawn like any other. **Later steps
are never drawn at random** — once the first step fires, the storyline is open
and the engine schedules the next step for `gap` terms later. It takes the
first of the term's two slots when due. A step whose branch flag was not set
(you cited the Hungarian, so the provenance paper never comes) is skipped; a
step whose rank band or stat gate is not yet met waits. A storyline ends when
its steps run out; it does not need a tidy ending.

Two events a term. A publication card is news and does not take a slot.

To add a storyline: write the events, give each later step a `ga.flag` for the
branch it belongs to (not `since` — timing is the storyline's job now), and add
one line to `ARCS`. Twenty-six exist.

## The cast

Recurring people have names, in `CAST`, and the prose refers to them with
placeholders: `{advisor}` for the full name and title, `{advisorS}` for the
surname alone. They are filled in when the card renders, so a name can be
changed in one place. In scenes and outcomes the name is clickable and opens
a short bio from `BIOS`, keyed like `CAST` without the `S`. **A new name in
`CAST` needs a `BIOS` entry** (a role line and a bio of three or four
sentences) or it renders as plain text.

| placeholder | who |
|---|---|
| `{worst}` | the title of the player's lowest-quality published paper — the one the prank caller asks about |
| `{qpaper}` `{qjournal}` | the paper under review longest and where it is — the one the process mods act on. **Every event gated `queue:1` must name it**, so the player can find it in the under-review list |
| `{paper}` | the title of the paper in the drawer — the one the next submission sends. Use it in a scene so the choice's `submit` is about a named paper |
| `{senior}` `{seniorS}` | Professor Ambrose Kettlewell — the senior man who offers his name for the paper (P4) |
| `{advisor}` | Professor Cornelius Vandersloot — has not read the chapter; wants his name on the job market paper; you write his obituary |
| `{discussant}` | Professor Dr. Dr. h.c. mult. Klaus-Dieter Frobenius — the theorem you extended; "confused"; a possible letter writer; on the Harwich longlist |
| `{star}` | Casimir Blunt — two papers in the Quarterly at thirty-four; the email; gives your talk better than you, later |
| `{calderon}` | Professor Aurelio Santangelo — replaces you at the whiteboard |
| `{student}` | Mira Halloran — the first student; the idea; Harwich; the inaugural lecture |
| `{rival}` | Yulia Sorokina — the Harwich job talk in your cohort; the reunion |
| `{laureate}` | Sir Alasdair Penhaligon-Brack — the oysters; his hand; his bad paper at the Review |
| `{editor}` | Gerald Fenn-Whistler — dinner, monthly, with everyone you are compared to |
| `{dean}` `{provost}` | Dean Constance Abara; Provost Leopold Marchand |
| `{admin}` | Bernadette — nine years; the petty cash |
| `{trustee}` | Chuck Kowalczyk III — logistics software; the resort; his roommate's fund |
| `{chair}` | Professor Ruth Eldridge — takes you for coffee in year six |
| `{postdoc}` | Lena Vogt |
| `{firm}` | Perpetua Vance, of Vance & Halloway — the search firm |
| `{hungarian}` `{historian}` `{prewitt}` `{chile}` | Ödön Székely-Bartha (1961); Dr. Winifred Scaife; Ottoline Prewitt, second-year with time; Joaquín Errázuriz, who read it |
| `{coauthor}` `{mostcited}` `{shouter}` | Dominic Fairweather, who stopped replying; Professor Gideon Marsh, most cited; Professor Rupert Coldstream, nine hours a year |

## Flags currently in play

Every flag set by an event is listed in `EPITHETS` in the source, with the line
it produces at the end. The ones other events currently gate on:

| flag | set by | gates |
|---|---|---|
| `GLOWING_SELF` / `HONEST_REF` | Y1, the journal sends you your own paper to referee (any paper under review) | `Y2G` the story at the bar / `Y2H` the editor's dinner |
| `LITIGIOUS` | Y3, threatening the Council | `Y4`, repeatable, a coin each time; a small penalty in every presidential search |
| `STATUE_DOWN` / `STATUE_PLAQUE` / `STATUE_KEPT` | Y5 | `Y6A` / `Y6B` / `Y6C` — the cameras, one version each |
| `EDGEWORTH_HOME` | Y8, the first edition | `Y9H` instead of `Y9` — the monument, with the book on your shelf |
| `CLASSROOM` | Y10, saying nothing | `Y11` — the dean's letter |
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
| `ADOPTED_MODEL` | F5 the central bank adopts it | F6 the recession |
| `CARTEL` | A6 the citation circle | K9 the ring, when a journalist maps it |
| `ASKED_SELF_CITE` | R1 | K10 the audit, as well as E3 the signature |
| `BET_GENIUS` → `GENIUS_LISTED` | Q16 the potential laureate | Q17 the call next door (coin), then Q18 the star's past |
| `WARNED_HIM` | G24 the spreadsheet | G25 heads up |
| `WATCHED` | G26 the laureate's hand | G27 the essay, eight terms later |
| `PROTECTED_STAR` | C7 the most cited member | C8 what the chair knew |

New flags are free — set one with `flag:"WHATEVER"`, gate a later event on it
with `ga:{flag:"WHATEVER"}`, and add a line to `EPITHETS` if it deserves to be
remembered at the end. Long-fuse consequences are the cheapest good thing in
the system.

---

## Papers

A career draws from a hundred handwritten titles, each with a field, without
replacement; a procedural generator is the reserve. Journals: the top four by
nickname, four generic A journals plus a field journal for each field, six B,
five C. An A-tier paper lands in its field journal about 60% of the time.

## Publication is an event

When a paper comes out, the term's card *is* the publication — title, journal,
tier, and a line that depends on the tier. It takes the slot an event would
have taken. Nothing to write for this; it happens in the engine.

## Endings that events can now reach

A Nobel-vow player who accepts a presidency (N14, or the Harwich committee in
G21) gets **The exit upward** — the office, the salary, and a phone that does
not ring for presidents. The vow is unmet, so the card closes on salary.

## What exists (221 events)

Events *available* at each rank, counting the ones whose band spans it:

| rank | available |
|---|---|
| PhD (0) | 18 |
| postdoc (1) | 36 |
| assistant (2) | 71 |
| associate / full (3–4) | ~155 |
| chair / dean / provost / president (office-gated) | 28 |
| about a field you published in | 19 — two per field |
| about citations | 12 |
| choice-free | 34 |

That is roughly the size the engine was built for. Adding more is still only
writing.

---

## Once, by default

Every event fires at most once per career. An event with `alt` texts can fire
once per text (`alt:[a,b]` → three times, the original first), and `max:n`
sets the count directly. `once:1` is now redundant and harmless. The draft
card draws its twelve scenes without replacement and falls back to one line
per band after that.

## Jokers

`JOKERS` is a third pool: the things one stumbles into. About one term in
eight, on top of the two events and never counted against them, a JOKER
card fires, weighted by `w`, each once per career. The common ones are big
strokes of luck or bad luck (the hit paper connected to nothing, the central
banker's speech, the textbook box, the medal, the bug found with
screenshots). The magical ones (`w:.1`, with a `when()` test on the state)
are the only way to Both: the laureate who is called by Harwich, the
president whose 2031 draft turns out to be the foundation. Add jokers as
`{id,ti,w,b,ga?,when?,sc,ch}`; they render as gold cards.

## The president's ladder

A president accrues a hidden `run` count: +1 for a choice that gains know-how
while in office, −1 for one that loses it, `run:n` in `fx` to set it
directly, a small drift each term from the know-how level. At `run` 4 and a
better college on the list, `PL1` The Better Offer fires (up to three times)
and taking it moves the presidency up a rung. Rungs and `run` warm the
Harwich search. `ga:{ladder:1}` is the gate; `{better}` names the college.

## Tenure

No tally. One probability from research, know-how, network, citations, top
four, profile, teaching debt and named enemies. Granted: associate professor
where you are. Denied: a question — a line at a lesser college with a
four-year clock (first denial only), an associate deanship (know-how 30),
or the exit. The Nobel is not killed by a denial any more; the lower
prestige does that work on its own.

## October

`OCTOBERS` is a pool of Nobel announcements, for players whose vow includes
the prize and who have not won it: on some falls (about half), on top of
everything else, once each, filtered by rank band. The young ones have
choices (the twist on page thirty-one, the joke written up seriously, slide
fourteen); the older ones are choiceless with a small effect, and the oldest
are about someone younger. Entries are `{t,b,s,ga?,fx?,ch?}`.

## Things one is sent to judge

`{ms}` draws a manuscript title from `MS_TITLES`, `{grant}` an application
from `GRANT_TITLES`, `{journalC}` a low journal; each is fixed for the card
it appears on, so the scene and the buttons agree. `MS1` The Manuscript
(three texts), `MS2` The Board, `GR1` The Panel use them. Add titles to the
pools freely.

## The literature

`LITERATURE` is a pool of other people's papers, noticed: choiceless dim
cards headed THE LITERATURE, once each, about one term in eight when no
Meanwhile card fired. `{topjournal}` `{ajournal}` `{bjournal}` `{cjournal}`
draw a journal of that tier, fixed per card. Entries are `{t,s,b,ga?,fx?}`;
gate the ones that should sting (`tdebt`, `nobelhope`, `E`, `since`).

**Reactions.** Any choiceless pool entry (Meanwhile, the literature, a
choiceless October) can carry `re:[["Read it.","one-line reply"],…]`: a row
of small buttons under the card that answer in a line and change nothing.
The term does not wait for them. Use them on some cards, not all.

## Meanwhile

`LIFE` is a separate pool of things that happen in a life and change
nothing: a nephew called Francis, an aunt's clock, the bacon charged twice.
One may appear in a term (about one in three), before the events, as a dim
card headed MEANWHILE, and it is never one of the term's two. Entries are
`{t, s, ga?, flag?}`: `ga` gates like an event, `flag` sets a flag so a later
entry can pick the thread up (`since:["NEPHEW",18]` is Francis at nine).
Each shows once per career. Most are free of effects; a dozen carry a small
ego lift (`fx:{E:6}`), because the ego otherwise only leaks. `{best}` names
the player's best paper in these.

**Multi-stage cards in one term.** A choice's `fn` can set `pendingNext` to
a function that opens the next card: `fn(){ pendingNext=()=>card(PRANK2);
return "…"; }`. The outcome then gets a Continue button, the slot is cleared,
and the next card opens at once. `Z4` → `PRANK2` → `PRANK3` is the prank
call, escalating within a single morning; the exits from the chain are
ordinary `fx` choices.

## Tense

The game runs term by term, so feedback is written in the present or the near
future: *it goes out; it will take a year; he says fourteen months*. A paper is
never announced as published in a choice's feedback — the publication card does
that when it happens. Retrospective jokes still work if framed as what you
already know: *you already know how the conference will go.*

## Notes

- `alt:[...]` on an event gives alternate scene text for repeat firings, so a
  recurring event doesn't read identically the third time. Worth it for
  anything without `once:1` that will fire often.
- Teaching is deliberately close to worthless. Events that make it matter
  should do so by *punishing neglect*, not by rewarding investment.
- The best events give the player a choice where both options are defensible
  and one of them quietly costs something that won't be visible for twenty
  years.
