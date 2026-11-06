# Run of show — slide by slide

The deck is `Voice_to_Action_Electronica_2026_Workshop - FINAL.pptx`, 24 slides.
Target ~45–50 min talk + demo, leaving room for Q&A. Read-aloud cues in my notes:
`/` = short pause, `//` = long pause, `[LIVE]` = do the thing, `[PAUSE]` = stop and let it land.

For each slide: **On screen** (what they see) · **Say** (the spine, not a script) ·
**Do** (clicks / demo) · **Watch for** (the thing that can go wrong or the question it provokes).

---

## Act 1 — WHY (slides 1–4)

### Slide 1 — Title: *Voice to Action*
- **On screen:** Title, my name, electronica 2026, Qualcomm Dragonwing.
- **Say:** One line of promise — *"In the next hour this device is going to learn two words it doesn't know yet, from your voices, and I'm going to ship that new skill to every board in the room over the air. Live."* Set the stakes, then move.
- **Do:** Nothing. Don't linger.

### Slide 2 — Agenda
- **On screen:** 5 sections — WHY / CHALLENGE / DATA & TRAINING / ARCHITECTURE / DEPLOY & VERIFY.
- **Say:** The hour is a single loop: understand *why the edge*, take on a concrete *challenge*, generate the *data* and *train*, see the *architecture* that makes it portable, then *deploy and verify* to the fleet. Promise the two live demos that bookend it.
- **Watch for:** Don't teach the agenda. 20 seconds.

### Slide 3 — Product → Platform (Dragonwing strategy)
- **On screen:** The strategy framing — moving from shipping a product to standing on a platform.
- **Say:** Qualcomm Dragonwing is the move from "a chip in a phone" to a platform for the intelligent edge — the same silicon lineage (CPU/GPU/NPU) now aimed at IoT, robotics, industrial. The workshop is a small, honest proof of that platform: a hobbyist-grade board and a production-grade board running *the same app*.
- **Watch for:** This is strategy, keep it to a minute; the proof comes later on slide 6 and 13.

### Slide 4 — Qualcomm Linux 2.0, the software foundation
- **On screen:** The software stack / QLI 2.0 foundation.
- **Say:** The thing that makes "prototype on a $X board, deploy on a production board" real is a *common, maintained Linux*. Qualcomm Linux 2.0 is upstream-first Yocto — the same build system, kernel lineage and package set across the family. That's what lets the model and the app travel. Plant the word **maintained** here; it pays off at slide 21.
- **Watch for:** "Upstream-first" and "Yocto" — if asked, slides 14 and the architecture doc back it.

---

## Act 2 — CHALLENGE (slides 5–7)

### Slide 5 — The challenge: add two words
- **On screen:** The device today knows some colours (purple / green / yellow); we're going to **add blue and red**.
- **Say:** Here's the concrete thing. The device recognises a handful of spoken colours and lights an LED to match. By the end it will also know **blue** and **red** — two words it has never heard — and it'll have learned them *from this room* and received the update *over the air*. No reflash, no cable.
- **Watch for:** This is the promise the whole talk is measured against. Say the two words clearly; they're the payoff on slide 20.

### Slide 6 — The hardware: two brains
- **On screen:** UNO Q and VENTUNO Q spec columns.
- **Say:** Both boards are **dual-brain**: a Linux applications processor *and* a real-time microcontroller on the same board. The big core runs the AI and the web app; the MCU drives the LEDs deterministically. **UNO Q** is the accessible prototyping board (QRB2210, quad Cortex-A53, STM32U585 MCU). **VENTUNO Q** is the production-class board (Dragonwing IQ-8275, octa-core Kryo, 16 GB LPDDR5, a Hexagon NPU at 40 TOPS, STM32H5F5 MCU). Same app, same model, two very different boards — *that's* the platform claim made concrete.
- **Do:** Point to the physical boards if they're on the table.
- **Watch for:** The NPU number and the MCU part numbers are the two things an engineer will probe. Numbers are in [03-hardware.md](03-hardware.md). (Our voice model runs on **CPU**, not the NPU — be ready for that; see the Q&A.)

### Slide 7 — Live demo, *before*
- **On screen:** Slide can go dark; the hardware is the screen now.
- **Say:** *"Let's see where it starts."*
- **Do:** `[LIVE]` Say **"select green"**, then **"select purple"** — LEDs follow. Then say **"select blue"** — *nothing happens*. *"It doesn't know blue. Yet. Let's fix that — with your help."*
- **Watch for:** This is the single most important 60 seconds. Mic check beforehand. If audio is flaky, see [06-demo-runbook.md](06-demo-runbook.md). The deliberate *failure* on "blue" is the setup — don't skip it.

---

## Act 3 — DATA & TRAINING (slides 8–11)

### Slide 8 — ML vs. traditional programming
- **On screen:** Rules you write (traditional) vs. examples you show (ML).
- **Say:** You can't write an `if` statement for "the sound of the word blue." Traditional code is rules in, behaviour out. Machine learning is examples in, a model out. We're not going to *program* the word blue — we're going to *show* the device enough examples of people saying it that it learns the pattern.
- **Watch for:** Keep it to the one contrast. Resist a tutorial.

### Slide 9 — Preparing the dataset
- **On screen:** What a good keyword dataset looks like — not just the colour words.
- **Say:** A useful dataset is more than the words you care about. Ours has **10 labels**: the colours, plus `select`, plus `noise`, `unknown`, and `update`. The negative classes matter as much as the positives — the model has to learn what *isn't* a command so it doesn't fire on a cough or a cellphone. Diversity of voices, distances, accents, background noise — that's what makes it robust in a loud hall like this one.
- **Watch for:** If asked "why `select`?" — it's a two-word grammar (`select <colour>`) that cuts false triggers. See the EI doc.

### Slide 10 — Your voice becomes the data
- **On screen:** Two QR codes — **BLUE on the left, RED on the right** — pointing at Edge Impulse data-collection links.
- **Say:** `[LIVE]` *"This is the part where you build it. Left half of the room — scan the left QR and say **blue** a few times. Right half — right QR, say **red**. Different distances, say it like you normally would. You are, right now, the training set."*
- **Do:** Hold for ~60–90 s of recording. Watch sample count climb if the EI project is on screen.
- **Watch for:** The QR links must be live and tested. **TODO: confirm the two collection URLs before the talk.** If the room can't scan, fall back to the pre-recorded dataset (runbook).

### Slide 11 — Start training
- **On screen:** Kick off training in Edge Impulse (impulse = MFE block → keyword-spotting transfer-learning block).
- **Say:** Now we train. The signal processing block turns each 1-second clip into a Mel-scale spectrogram — a picture of the sound. The learning block is a small neural net, pre-trained on keyword spotting, that we fine-tune on *our* words. It trains in a couple of minutes because we're standing on transfer learning, not starting from zero.
- **Do:** Start the training job. It runs in the background while I talk through architecture (slides 12–15).
- **Watch for:** Training time is the hidden clock. Start it, then *move on* — don't watch the bar. Details in [04-edge-impulse.md](04-edge-impulse.md).

---

## Act 4 — ARCHITECTURE (slides 12–15)  *(training runs in the background)*

### Slide 12 — Two containers
- **On screen:** `webapp-led-mcu-voice` (the app) and `webapp-led-mcu-voice-assets` (the model delivery) sharing a volume.
- **Say:** The app is split in two on purpose. One container is the **application**: it reads the mic, runs the model, lights the LEDs, serves the web UI on port 8000. The second container's only job is to **deliver the model file** (`deployment.eim`) into a shared volume and nudge the app to reload it. Why split them? Because retraining is then just *replacing a file* — an app-level update — not rebuilding the OS.
- **Watch for:** This split is the hinge of the whole deploy story. Make it stick.

### Slide 13 — Portability and the hardware boundary
- **On screen:** What travels (the model, the app) vs. what's board-specific (the hardware access).
- **Say:** The model and the Python are identical on both boards. What changes is the thin boundary where software meets hardware: which ALSA device is the mic, which serial port talks to the MCU. Draw that boundary cleanly and the same artifact runs on a $X prototyping board and a production board. *That's* portability — not a slogan, a file you can point at.
- **Watch for:** Great place to reference the containers again. The mic is the most visible boundary (the demo literally depends on it).

### Slide 14 — Yocto defines the image
- **On screen:** Yocto / the OS build.
- **Say:** Under the containers is an OS we actually control. Yocto lets us define, pin and rebuild the exact image — kernel, drivers, the container runtime — reproducibly. That's the difference between "it works on my board" and "I can rebuild this image in two years for a security fix." Upstream-first means we're not maintaining a fork forever.
- **Watch for:** This is where CRA / long-term maintenance questions land. Have the answer from [07-anticipated-qa.md](07-anticipated-qa.md).

### Slide 15 — Export `deployment.eim`
- **On screen:** Edge Impulse → *Deploy* → Linux (AARCH64) → the `.eim` file.
- **Say:** Training's done. We export the model as a single self-contained file — a `.eim` — built for 64-bit Arm Linux. One artifact, runs the same on both boards. This file is the thing we're about to ship.
- **Do:** If training finished, show the Deploy screen and the download. This is the baton-pass from the ML half to the deploy half.
- **Watch for:** Pick **Linux (AARCH64)**, the CPU/unoptimized build — *not* a QNN/NPU target. If asked why, see the Q&A (short version: the CPU build "just runs," the NPU build needs delegate libs we don't mount, and for 10 keywords the CPU is plenty).

---

## Act 5 — DEPLOY & VERIFY (slides 16–21)

### Slide 16 — A maintained update path (TUF)
- **On screen:** Signed update metadata; an example target JSON (RB3 Gen2).
- **Say:** Shipping a file to a fleet safely is its own discipline. We use **TUF** — The Update Framework — so every update is signed and versioned, and a device will *refuse* anything that isn't properly signed or that tries to roll it backwards. This is the boring, essential part that separates a demo from a product.
- **Watch for:** Security people perk up here. Know what TUF protects against (see Q&A: rollback, key compromise, mix-and-match).

### Slide 17 — A commit starts the pipeline
- **On screen:** `git add / commit / push`.
- **Say:** How do I trigger a fleet update? I commit. The new model file lands in git, a build picks it up automatically, and a new **target** — a signed, versioned description of "this is the software a device should run" — gets produced. The deploy is driven by the same workflow I already use to write code.
- **Watch for:** Ties the dev loop (slide 22, AI-assisted) to the ops loop. "Push to deploy" is the soundbite.

### Slide 18 — The local server coordinates the rollout
- **On screen:** The self-hosted update server / its web UI.
- **Say:** Everything runs **on-prem** — the update server lives on the VENTUNO Q itself, no cloud dependency. I create a **rollout** targeting the device group, and the boards, which check in every few seconds, pick it up. I can watch it happen.
- **Do:** Open the server UI, create / show the rollout to the `demo` group.
- **Watch for:** "Why self-hosted?" — sovereignty, air-gap, no per-device cloud cost. Answer in Q&A.

### Slide 19 — Fleet view
- **On screen:** The devices list / fleet dashboard.
- **Say:** Here's the fleet. Each board reports what target it's running. I can see the update propagate from "offered" to "installed" across the group, live. This is the verify half of "deploy and verify."
- **Do:** Show the device(s) moving to the new target.

### Slide 20 — Live demo, *after*  ← THE PAYOFF
- **On screen:** Dark. Hardware again.
- **Say:** *"The device just received an update it didn't have when we started. Let's ask it the question that failed."*
- **Do:** `[LIVE]` Say **"select blue"** → blue lights. Say **"select red"** → red lights. `[PAUSE]` *"Twenty minutes ago it had never heard those words. You taught it, and we shipped it — over the air, to the whole room."*
- **Watch for:** This is the moment the talk exists for. If the fresh model is risky live, I can have a verified fallback model staged — see the runbook. Land the pause.

### Slide 21 — App and OS updates have different jobs
- **On screen:** Two independent update lanes — application vs. operating system.
- **Say:** What you just saw was an **app update** — a new model file, no reboot, seconds. The *OS* updates on a separate track: atomic, signed, reboot-into-the-new-image, roll back if it fails to boot. Decoupling them is why I can retrain ten times a day without ever touching the base system — and still ship a kernel security fix when I need to. That's maintainability.
- **Watch for:** This is the intellectual payoff of the whole architecture. The "retrain = file swap, not OS rebuild" line from slide 12 closes here.

---

## Act 6 — CLOSE (slides 22–24)

### Slide 22 — Agentic AI in the build
- **On screen:** Claude Code / AI-assisted development.
- **Say:** Honesty about how this got built: an AI coding agent wrote and wired a lot of this — the containers, the glue, the server config — against real hardware, under my direction. The judgment, the architecture, the "no, that's wrong" — that stayed human. The point isn't that AI replaces the engineer; it's that the loop from idea to running-on-hardware got dramatically shorter.
- **Watch for:** Don't oversell. If asked "what did *you* do vs the AI," answer plainly (Q&A).

### Slide 23 — Prototype → Production
- **On screen:** The lifecycle — SELECT / PROTOTYPE / OPTIMIZE / PRODUCTIZE / MAINTAIN / SCALE.
- **Say:** Zoom out. Everything today maps to a real product lifecycle: *select* the platform, *prototype* on the accessible board, *optimize* the model, *productize* on the production board, *maintain* over the air for years, *scale* across a fleet. The workshop is a one-hour compression of a multi-year journey — and the same tools carry you the whole way.
- **Watch for:** This is the "so what / now what." Tie each stage back to a thing they saw.

### Slide 24 — Thank you + repos
- **On screen:** Two QR codes: `github.com/munoz0raul/electronica-voice-workshop` and `github.com/munoz0raul/voice-workshop`.
- **Say:** *"Everything you saw is open. One repo is the ML and the app — clone it, `docker compose up`, add your own colour. The other is the OTA and fleet side. Go teach something a new word."* Thanks, invite questions.
- **Watch for:** Have the Q&A doc loaded in my head. Leave the repos on screen during Q&A.
