# Edge Impulse — the ML half, end to end

This mirrors the reproduction documented in
[munoz0raul/voice-workshop `docs/edge-impulse.md`](https://github.com/munoz0raul/voice-workshop/blob/main/docs/edge-impulse.md)
and adds the *why* behind each choice so I can defend any of it from the stage.

**What we're building:** a keyword-spotting model that listens to 1-second
windows of 16 kHz audio and classifies each into one of 10 labels. Exported as a
`.eim` for 64-bit Arm Linux, it drops into the app's shared volume and the app
starts acting on the new words. Retraining → new `.eim` → ship = the whole deploy
story.

## The labels (10)

```
blue  green  orange  purple  red  yellow      ← the six colours
select                                         ← the command verb
noise  unknown  update                         ← the negative / control classes
```

The app only *acts* on six of them: `select` plus the five colours it's wired to
(`blue green purple red yellow`). The grammar is **`select <colour>`** — a
two-word trigger that sharply cuts false fires compared to a bare colour word.

**Why the negative classes exist** (this is the question a thoughtful person
asks): `noise`, `unknown`, and `update` teach the model what is *not* a command.
Without them, a model trained only on colour words will confidently shout "green"
at a cough, a chair scrape, or an unrelated word, because those are the only
options it has. Negative classes give it an "I didn't hear a command" answer.

## The pipeline in Edge Impulse (8 steps)

1. **Create a project** — type: audio / keyword spotting.
2. **Data acquisition** — collect labelled 1-second clips at **16 kHz**. On the day
   this is the audience via the two QR links (blue, red). The base dataset is
   pre-recorded and lives on the voice-workshop repo's Releases
   (`edge-impulse-voice-detection-export.zip`, ~100 MB).
3. **Upload / import the dataset** — bring in the existing labelled clips so we're
   adding to a solid base, not starting from the room's noisy samples alone.
4. **Verify samples** — spot-check that clips are labelled right and are ~1 s.
5. **Create the impulse** — window **1000 ms**, stride, 16 kHz input, then two
   blocks:
   - a **processing block: Audio (MFE)** — Mel-filterbank energies,
   - a **learning block: Transfer Learning (Keyword Spotting)**.
6. **Generate features** — run the MFE over every clip to produce the spectrogram
   features the net trains on. (Good moment to show the feature explorer — clusters
   of each word are visibly separable.)
7. **Train** — fine-tune the pre-trained keyword net on our features. Minutes, not
   hours, because of transfer learning. Watch accuracy / confusion matrix.
8. **Deploy** — *Deploy* → **Linux (AARCH64)** → download the **`.eim`**.

## Why MFE (not MFCC, not raw audio)

- **Raw waveform** into a small net is wasteful — the net would spend capacity
  relearning basic time-frequency structure.
- **MFE (Mel Filterbank Energies)** produces a spectrogram-like image on a
  perceptual (Mel) frequency scale that matches how speech energy is distributed.
  It's the sweet spot for small keyword models: compact, robust, and the
  pre-trained transfer-learning block expects this kind of input.
- **MFCC** adds a cepstral (DCT) step on top of MFE — historically great for
  classical speech pipelines, but for small CNN-based keyword spotting the extra
  decorrelation usually isn't needed and MFE keeps more of the spectral detail the
  net can use. Edge Impulse's own keyword-spotting guidance leans MFE for this
  reason. Short version on stage: *"MFE turns each clip into a picture of the sound
  on a perceptual scale; the net is good at pictures."*

## Why transfer learning (not train from scratch)

- A keyword net trained from zero needs a lot of data and time. We have a room and
  a few minutes.
- Edge Impulse's **Transfer Learning (Keyword Spotting)** block starts from a net
  already trained on a large speech corpus and fine-tunes the last layers on our
  10 labels. It converges in minutes on a modest dataset and generalises better to
  unseen voices — exactly the live-demo constraint.
- This is *the* reason the live retrain is even plausible in a one-hour slot.

## Why the `.eim` and why AARCH64 / CPU

- A **`.eim`** is Edge Impulse's self-contained Linux deployment: the model plus
  its signal-processing and a small runner, in one executable file the app loads.
- We build it for **Linux (AARCH64)** so the *same* file runs on both 64-bit Arm
  boards (UNO Q and VENTUNO Q) — one artifact, no per-board rebuild.
- We use the **CPU / unoptimized (float32)** build, **not** a QNN/NPU-accelerated
  one. Two reasons: (1) the model is tiny, the CPU handles it in real time; (2) the
  CPU `.eim` has no external accelerator dependencies, so it "just runs" in our
  container, whereas the QNN build needs Qualcomm delegate libraries mounted into
  the container — a landmine that makes the model hang and return all-zero scores
  if they're absent. Portability + reliability win over peak speed for 10 keywords.
  (This exact gotcha bit the build; it's in the Q&A.)

## "Add a colour" — the home-exercise version

The voice-workshop README walks through adding a new colour: add a label, collect
samples for it, retrain, export a new `.eim`, drop it in. This is the audience's
takeaway — the live demo is the same loop, compressed and shipped over the air
instead of copied locally.

## Runtime knobs the app applies to the model output

Set in the app's compose env (so I can explain *how a raw classifier becomes a
reliable trigger*):

| Env | Value | What it does |
|---|---|---|
| `THRESH` | `0.70` | Minimum confidence before a word counts — rejects weak guesses. |
| `DEBOUNCE_SECONDS` | `0.5` | Ignore repeat detections within this window — one word, one action. |
| `SELECT_COOLDOWN_SECONDS` | `1.0` | After `select`, brief wait before accepting the colour. |
| `SELECT_SUPPRESS_SECONDS` | `8.0` | After a completed `select <colour>`, suppress re-triggering. |

These are *post-processing*, not the model — good to show that a shippable voice
feature is "model + a little grammar and debouncing," which is honest engineering.

## TODO before the talk
- Confirm the two **data-collection QR URLs** (blue / right, red / left) are live.
- Confirm the **public EI project URL** (voice-workshop's `dataset/README.md` still
  lists it as `TODO`); have it ready if someone asks to see the project.
