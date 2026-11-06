# Voice to Action — Presenter Study Pack

Everything I need to walk on stage at **electronica 2026** and present the
*Voice to Action* workshop as the person who built it — not read the slides, but
answer any question the room throws.

This repo is **study material**, not demo code. The runnable demo and the full
reproduction docs live in the two public repos the deck points at:

- **[munoz0raul/electronica-voice-workshop](https://github.com/munoz0raul/electronica-voice-workshop)** — the OTA / fleet / Yocto side (self-hosted update server, how the OS and apps are built and rolled out).
- **[munoz0raul/voice-workshop](https://github.com/munoz0raul/voice-workshop)** — the app + Edge Impulse side (`docker compose up`, the keyword model, "add a colour").

## The one sentence

> We teach a device two new words with our voices, and ship that new skill to a
> fleet over the air — the whole loop from *data* to *deployed*, live, in one hour.

## How to use this pack

Read in this order the week before; skim the first two the morning of.

| # | File | What it gives you |
|---|------|-------------------|
| 1 | [docs/01-run-of-show.md](docs/01-run-of-show.md) | Slide-by-slide talk track: what's on screen, what I say, what I click, timing. |
| 2 | [docs/02-key-messages.md](docs/02-key-messages.md) | The thesis and the 4 messages everything ladders up to. The lines to land. |
| 3 | [docs/03-hardware.md](docs/03-hardware.md) | UNO Q / VENTUNO Q / IQ-9075 — specs, the dual-brain idea, why it matters. |
| 4 | [docs/04-edge-impulse.md](docs/04-edge-impulse.md) | The ML half end-to-end: dataset → MFE → transfer learning → `.eim`. Reproducible. |
| 5 | [docs/05-architecture.md](docs/05-architecture.md) | Two containers, the hardware boundary, Yocto, the self-hosted OTA server, fleet view. |
| 6 | [docs/06-demo-runbook.md](docs/06-demo-runbook.md) | The live demo mechanics and exactly what to do when something misbehaves. |
| 7 | [docs/07-anticipated-qa.md](docs/07-anticipated-qa.md) | The question bank — the hard ones, with answers I can defend. |
| 8 | [docs/08-glossary.md](docs/08-glossary.md) | Every acronym and term in the deck, one line each. |
|   | [CHEATSHEET.md](CHEATSHEET.md) | One page. Print it. Specs, env values, demo phrases, panic buttons. |

## The shape of the hour

```
WHY          →  CHALLENGE     →  DATA & TRAINING  →  ARCHITECTURE    →  DEPLOY & VERIFY
(strategy)      (add 2 words)    (you record it)     (containers/OS)    (OTA to the fleet)
 slides 3-4      5-7              8-11                12-15              16-21  → payoff 20
```

Demo bookends it: **demo-before** (slide 7, device knows the old words) and
**demo-after** (slide 20, device now knows *blue* and *red* because we just
trained and shipped it). Everything between is how that happened.

## Honesty about sources

Technical claims here trace to the two demo repos, the app source, the deck, or
the build work behind the demo. Where a number or path is load-bearing I kept it
exact. Where something is my framing for the stage rather than a spec, it reads
as framing. Anything still open is marked **TODO**.
