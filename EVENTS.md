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
| `ch` | the choices, 1–4 of them |

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
- `flag:"NAME"` — sets a permanent flag; add it to `EPITHETS` to have it appear
  in the ending's *"The field remembers…"* line
- `foe:1` — makes an enemy, with a generated name, grudge and what they poison
- `mod:"name"` — a one-shot engine modifier
- `ending:"EXIT"` — ends the career immediately

**`roll`:** `stat` is one of `R` `K` `N` `P` `E`; `pivot` is the value at which
it's a coin flip (default 50). Ego shifts the odds in your favour, scales the
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

A weight of `1` is a normal pull, `2` a strong one. Every draw also has a
"quiet term" weight in the pot, so nothing is guaranteed.

---

## Gates (`ga`, and per-choice `gate`)

`{R:45}` `{K:50}` `{N:40}` `{P:60}` `{E:70}` `{flag:"SPEC"}` — any combination.
On an event, an unmet gate means it can't fire. On a choice, the button is
shown disabled with a reason, which is often better than hiding it: seeing what
you can't do yet is part of the picture.

---

## Flags currently in play

`RAN_SEMINAR` `ANTISOCIAL` `GENEROUS` `HONEST` `SPEC` `CORRECTED` `CARTEL`
`OUTSIDE` `MENTOR` `THEPAPER` `EDITOR` `PHONED_IN` `AGENT` `VIRAL` `CHOSE_LIFE`
`TRUTHFUL` `THURSDAYS` `GOODHIRE` `CANDID` `LOYAL` `TWOBODY` `FUNDED`
`ADMIN_ESCAPE` `FEST` `BORROWED` `TENURED` `DENIED_ONCE` `POSTDOC`

New flags are free — set one with `flag:"WHATEVER"`, gate a later event on it
with `ga:{flag:"WHATEVER"}`, and add a line to `EPITHETS` if it deserves to be
remembered at the end. Long-fuse consequences are the cheapest good thing in
the system.

---

## What exists (33 events)

| stage | have | thin on |
|---|---|---|
| PhD (0) | 6 | the advisor relationship, cohort rivalry, the market itself |
| postdoc (1) | 3 | almost everything — this stage is nearly empty |
| assistant (2) | 7 | the clock's psychology, coauthors, the first PhD student |
| associate/full (3–4) | 11 | the long middle, editorships, decline |
| admin track | 5 | deans, budgets, trustees, protests, fundraising |
| any stage | 6 | money, family, health, the world outside |

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
