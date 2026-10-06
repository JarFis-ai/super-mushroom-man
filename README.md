# Super Mushroom Man - case study

**[Read the full case study →](https://jarfis-ai.github.io/super-mushroom-man/)**

Super Mushroom Man is a projector-first team quiz for ESL classrooms. The teacher opens it
on a laptop, projects it to a screen, splits the class into teams, and runs it. Students
answer out loud and never touch a device.

The hook is that the points are unstable. A correct answer never just pays a fixed score -
it opens a block, and that block might be worth one point, might be worth seven, might
double your total, or might hand every point you own to another team. A team in last place
is never out of it, which is the only reason the format holds a class of ten-year-olds for
twenty minutes.

It is built for hagwons and academies in Korea: 2 to 8 teams, students roughly 7 to 16.

This repository is the case study and screenshots. The application source is private.

---

## Where it came from

A 54-slide fan-made PowerPoint that has been passed between ESL teachers for over a decade.
A grid of lettered blocks, each hyperlinked to a question, each question hyperlinked to a
reward slide. A genuinely good classroom format held together entirely by hyperlinks.

What it had never had was an engine, a scoreboard, or artwork anyone was allowed to use.

| The PowerPoint | The rebuild |
|---|---|
| Rewards hand-wired into 24 hyperlinks | Shuffled from a seeded bag at game start. Zero teacher prep |
| Scores kept on a whiteboard, by hand, mid-lesson | A live scoreboard, with the engine applying every outcome |
| Your own questions meant editing the deck | Pick a built-in pack and a level, or import a saved list |

I read the deck's own hyperlinks to recover its real reward mix rather than guessing at one.
It came out at 14 coin payouts to 10 mystery blocks, which is far more chaotic than I would
have designed from scratch. My own earlier guess had been 17 to 7. Ten years of real
classrooms had already tuned it better than I would have.

## The rule that shapes everything

**Every scoring decision is a pure function.** No React, no `Date.now()`, no `Math.random()`,
no DOM, no network. One reducer:

```ts
applyOutcome(state: GameState, action: GameAction): GameState
```

Randomness enters only through a seeded generator whose state lives inside the game state,
so any game is reproducible from its seed. That is what makes the chaos outcomes testable
at all: "swap with last place when three teams are tied for last" is a unit test, rather
than a report from a teacher who cannot quite remember what happened.

It is also why the answer timer lives in a component and not in the engine.

## The one that is easy to get wrong

Rewards are **dealt from a bag, not pinned to blocks.**

Two fairness rules keep the game from turning cruel: never deal a mystery block as the very
first reward of the game, and never deal two in a row. Both are rules about the order
rewards are *dealt*. But the teacher chooses the order blocks are *opened*, and a wrong
answer pays nothing at all, so board order is not deal order and never will be. The bag is
shuffled and arranged once at game start, and the guards then hold for the whole game
whatever order the class picks.

## Built with

Next.js 15 (App Router) · React 19 · TypeScript strict · Tailwind v4 · Vitest · Vercel

Three runtime dependencies, and no state, animation or UI library. Every sound is
synthesised from oscillators at runtime, so the repository contains no audio files.

## Where it stands

| | |
|---|---|
| Tests | 130 passing across 6 files |
| Mystery outcomes | 7, each with its own edge-case tests |
| Questions | 270, in 3 packs at 3 levels each |
| Audio files | 0. Every cue is synthesised |
| Runtime dependencies | 3 |
| Remaining work | Platform integration, blocked on the platform rather than on this code |

## Three things that were not obvious up front

1. **Sizing from viewport width was wrong.** The first layout fitted a wall-mounted TV
   perfectly and overflowed a laptop, because the browser takes a strip for tabs and Windows
   takes another for the taskbar. Every class-facing size is now a multiple of one unit,
   `min(1vw, 1.9vh)`, and the fit problem stopped coming back.
2. **A tie can look exactly like a bug.** An early draft had "swap with last place" pick the
   first tied team and name it, so a team on three points could swap with another team on
   three points and read "Team 1 and Team 2 swap points" while the scoreboard did not move.
   From ten metres away that is indistinguishable from a broken scoreboard. Being level with
   the lowest score now means you *are* last, and the screen says so.
3. **Writing rules that live only in a document decay.** The content rules are a test file.
   It fails the build on an em dash, on a word from the scary list, on a prompt over 110
   characters, on a duplicate prompt, on an empty answer, on a level with no open-ended
   question, and on a level whose prompts do not measurably lengthen as difficulty rises.

---

Built by [Jacobus Barnard](https://github.com/JarFis-ai), a teacher and developer in Seoul.
Also: [59 Seconds](https://jarfis-ai.github.io/), a live ESL speaking game, and [Taco Trivia](https://jarfis-ai.github.io/taco-trivia/).
