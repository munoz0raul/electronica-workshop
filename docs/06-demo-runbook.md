# Demo runbook — mechanics and fallbacks

Two live moments carry the talk: **demo-before** (slide 7) and **demo-after**
(slide 20). Everything else can wobble; these two must land. This is the "what do I
press and what do I do when it misbehaves" doc.

## Pre-flight (do this before the room fills)

- [ ] Boards powered, booted, on the demo Wi-Fi, and reachable.
- [ ] Mic connected and selected as the **right capture device** (the hardware
      boundary — wrong device = silent model). On the UNO Q that's ALSA device 3.
- [ ] App up: web UI reachable on **:8000**; it shows what it's hearing.
- [ ] Say "select green" / "select purple" once off-mic-to-audience to confirm
      recognition and LED response.
- [ ] Update server UI (**:8080**) open on a browser tab, logged in, fleet visible.
- [ ] Edge Impulse project open, logged in; the two **data-collection QR links
      tested from a phone** (scan → record → sample appears). **TODO: confirm URLs.**
- [ ] A **known-good fallback `.eim`** staged and ready to ship, in case the
      live-trained model is too noisy to trust (see "Fallback A").
- [ ] Decide the room split for slide 10 (left = blue? right = red? — match the
      QR layout on the slide: **BLUE left, RED right**).

## The two live beats

**Slide 7 — before.** Say "select green" (lights green), "select purple" (lights
purple), then **"select blue" → nothing**. Narrate the gap: it doesn't know blue
yet. The non-event is the whole setup; don't rush past it.

**Slide 20 — after.** "select blue" → blue. "select red" → red. Pause. Deliver the
"you taught it, we shipped it" line. If either colour misfires once, calmly repeat
the phrase once — a single retry reads as normal voice UX, not failure.

## The hidden clock: training time

Training starts on slide 11 and must finish before slide 15 (export) / 20 (demo
after). That's the ~15–20 minutes of architecture talk (slides 12–15) plus deploy
(16–19). If training is slow:

- Keep talking — the architecture act is designed to fill exactly this gap.
- Have the **fallback `.eim`** ready so the *ship* step (the real message) happens
  on time even if the fresh model isn't ready or isn't good.

## Decision tree when something breaks

### Mic / recognition dead at slide 7
1. Check the capture device selection first (most common cause).
2. Check the app container is up and the UI on :8000 shows audio activity.
3. If unrecoverable: pivot to the web UI as the input — show detections there, or
   narrate from a recorded clip. Don't stall on it; the message survives.

### Room can't collect data at slide 10 (bad Wi-Fi, QR won't scan)
- Fall back to the **pre-recorded dataset** (voice-workshop Releases,
  `edge-impulse-voice-detection-export.zip`). Say: *"We've also got hundreds of
  samples recorded earlier — let's train on those"* — the loop is identical, the
  audience just didn't add to it. The message (edge learns from data) is intact.

### Training stalls or finishes with poor accuracy
- **Fallback A (preferred):** ship the pre-staged known-good `.eim` instead of the
  freshly trained one. The *deploy over the air* is the payoff; the audience can't
  tell which model binary shipped, and "select blue/red" still works.
- Be honest if asked: live training in a loud hall is hard; the point is the loop
  and the OTA path, which are real either way.

### OTA rollout doesn't land on slide 20
1. Check the device checked in (fleet view, :8080) and the rollout targets the
   right group (`demo`).
2. App updates are seconds; if it's not there, it's usually the rollout not created
   or wrong group — recreate it.
3. **Do NOT** start deleting/recreating targets in the UI under pressure — that can
   trip TUF's anti-rollback and brick the update ("rollback attempt"); recovery
   needs clearing the device's cached metadata, which is not a stage move. If it
   won't land, fall to Fallback B.
- **Fallback B:** the model is already in the shared volume path on the board; a
  local restart of the app with the new `.eim` present demonstrates the *same*
  before/after. Frame it as "here it is on this board" — still a real
  learn-then-act, minus the fleet flourish.

### A single colour misfires live
- Repeat the phrase once, clearly. One retry is invisible. If it persistently
  misfires, switch to a colour that *does* work and move on — don't debug on stage.

## What I will not do on stage
- Reflash a board, edit server targets by hand, or touch the OS update lane. All
  slow and/or fragile; none of it is the message.
- Chase perfect recognition. The loop is the point.

## Restart cheats (only if I have shell access and time)
- Reload the voice app on a new model: the assets flow restarts it with **`kill -9`**
  (SIGTERM is dropped inside the app's PID namespace — a plain restart won't pick up
  the new `.eim`).
