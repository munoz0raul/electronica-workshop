# Anticipated Q&A — the question bank

The questions a sharp audience asks, with answers I can defend. Grouped by theme.
Each answer is a *spoken* answer — short, honest, with the depth ready if they
push. Where the true answer is "it depends" or "I'd check," it says so; a confident
wrong answer costs more than an honest "let me confirm."

---

## A. Machine learning / Edge Impulse

**Q: Does this run on the NPU? You've got 40 TOPS sitting there.**
No — the keyword model runs on the **CPU**. Two reasons. First, it's a tiny model:
10 keywords, 1-second windows; the CPU does it in real time with room to spare.
Second, I build the model as a portable CPU `.eim` so the *same file* runs on the
UNO Q, which has no NPU, and on the VENTUNO. The 40-TOPS Hexagon is the platform's
headroom — it's why you'd productize on this board when your next model is vision
or a much bigger audio net. For *this* task, using it would be engineering theatre.

**Q: So why not show the NPU doing something?**
Because the honest demo is the one I can ship live. The CPU build has no external
dependencies; the NPU build needs Qualcomm delegate libraries mounted into the
container, and if they're missing the model silently hangs and returns all-zero
scores. For a one-hour live session, reliability beats a benchmark. The NPU story
is the *next* slide of a product's life, not this one.

**Q: How much data do you actually need to add a word?**
Not much, because of **transfer learning** — we fine-tune a net already trained on
a large speech corpus, so tens-to-low-hundreds of clips per word gives usable
accuracy. From scratch you'd need orders of magnitude more. That's the entire
reason a live retrain fits in the talk.

**Q: Why MFE and not MFCC or raw audio?**
MFE turns each clip into a Mel-scale spectrogram — a perceptual picture of the
sound — which is what a small CNN is good at. MFCC adds a cepstral transform on top
that classical pipelines liked but that small keyword nets usually don't need; MFE
keeps more spectral detail. Raw audio wastes model capacity relearning basic
time-frequency structure. MFE is the sweet spot for this block.

**Q: What are those extra labels — noise, unknown, update?**
Negative/control classes. They teach the model what *isn't* a command so it doesn't
fire "green" at a cough or a stray word. A model trained only on the words you want
is overconfident on everything else, because those are its only options. In a hall
this loud, the negatives are doing real work.

**Q: Why the two-word "select blue" instead of just "blue"?**
It's a deliberate grammar to cut false triggers — a single common word fires far
more often by accident. Requiring `select` first, with a short cooldown, means
casual conversation in the room doesn't keep flipping LEDs. It's the cheap version
of a wake-word.

**Q: How do you handle the noise in a room like this?**
Three layers: diverse training data (including noise clips), a **confidence
threshold** (0.70 — weak guesses are dropped), and **debouncing** (0.5 s) plus a
`select` cooldown so one utterance is one action. The model alone isn't the
product; the model plus that little grammar is.

**Q: What's the latency / window?**
1-second analysis window at 16 kHz. Perceived response is near-immediate — well
under what feels laggy — because inference is tiny on the CPU and the MCU renders
the LED with real-time timing.

**Q: Could I use a different framework — TensorFlow Lite, ONNX?**
Yes; Edge Impulse is the on-ramp, not a lock-in. It exports standard artifacts, and
the `.eim` is just a convenient self-contained runner. The architecture — model as
a swappable file delivered by a sidecar container — doesn't care what trained it.

---

## B. Hardware

**Q: Why two microcontrollers — isn't the Linux SoC enough?**
It's one MCU *per board*, alongside the Linux SoC — that's the dual-brain design.
LEDs (an 8×13 matrix plus RGB) need deterministic timing; you don't want a Linux
scheduler hiccup glitching the display. So Linux does intelligence and
connectivity, the Cortex-M33 does the real-time IO. It's the same split you see
from a doorbell to a robot arm.

**Q: What's the real difference between UNO Q and VENTUNO Q then?**
Class. UNO Q (QRB2210, quad A53, LPDDR4) is the accessible prototyping board.
VENTUNO Q (Dragonwing IQ-8275, octa-core Kryo, 16 GB LPDDR5, a 40-TOPS Hexagon NPU,
2.5 GbE) is production-class with real headroom. The point of the demo is that the
*same app and model* run on both — you don't rewrite when you move from bench to
product.

**Q: Which MCU is on which board?**
UNO Q: STM32U585, Cortex-M33 @ 160 MHz. VENTUNO Q: STM32H5F5, Cortex-M33 @ 250 MHz.
I verified this doing the firmware work — the VENTUNO's Zephyr firmware is built
for the H5.

**Q: How does Linux talk to the MCU?**
Serial — an LPUART at 115200 — using MCUmgr/SMP with a custom command group. The
app sends a compact "render these LEDs" command; the Zephyr firmware executes it.
The MCU firmware itself is part of the OS image and loaded at boot from the Linux
rootfs, which is why a firmware fix is an OS-lane change, not an app update.

---

## C. Architecture / containers

**Q: Why two containers instead of one app?**
To make the model *data* rather than *code*. One container is the app; the other's
only job is to deliver the model file into a shared volume and trigger a reload.
Retraining then changes only that file — an app-level update, seconds, no reboot —
instead of rebuilding an image. The split is what makes "retrain all day" cheap.

**Q: What exactly is portable, and what isn't?**
Portable: the `.eim`, the app code, the container images, the web UI — byte-for-byte
the same on both boards. Not portable: the thin hardware boundary — which ALSA
device is the mic, which serial port is the MCU. Keep that boundary in config and
the artifact travels. The mic is the boundary you can literally see in the demo.

**Q: Isn't Docker on an embedded device heavy?**
It's a deliberate trade: containers buy you the clean app/OS separation and the
fast app-update lane, on boards that have the RAM and storage for it (the VENTUNO
has 16 GB). For deeply constrained MCUs you wouldn't; for a Linux-class edge device
aimed at maintainability, the isolation and update story pay for the overhead.

---

## D. Deployment / OTA / security

**Q: Is this cloud-based? What's the dependency?**
No cloud. The update server runs **on-prem, as a systemd service on the VENTUNO Q
itself** — web UI on :8080, device gateway over mutual TLS on :8443. Air-gap it and
it still works. That's a real requirement for industrial, defence, and
data-sovereignty customers.

**Q: How do you stop a malicious or corrupt update?**
**TUF — The Update Framework.** All update metadata is signed; devices reject
anything unsigned or mis-signed, and reject anything *older* than what they run, so
you can't be rolled back to a vulnerable version. Combined with mutual-TLS between
device and server, a device only installs software that's both authentic and
forward-moving.

**Q: What actually triggers a deployment?**
A git commit. The new model lands in the repo, an automatic build picks it up and
produces a new signed **target** — a versioned description of the complete software
a device should run. I create a **rollout** to the device group; boards pick it up
on their next check-in, every few seconds. Push-to-deploy, same workflow as writing
code.

**Q: App update vs OS update — what's the difference?**
Two independent lanes. App update: swap a compose app / the model file, no reboot,
seconds, roll back by redeploying. OS update: a new atomic ostree image, reboot
into it, and it falls back automatically if the new image won't boot. Decoupling
them means I retrain endlessly without touching the base OS, yet can still ship a
kernel CVE fix on the OS lane. That decoupling is the maintainability thesis.

**Q: What if an OS update bricks the device?**
ostree deployments are atomic and keep the previous one. If the new image fails to
boot, the bootloader falls back to the last good deployment. That's the difference
between an atomic image update and `apt upgrade` on a device you can't physically
reach.

**Q: Have you actually seen the anti-rollback bite?**
Yes — and it's instructive. Deleting targets by hand in the server UI regressed the
generated TUF version, and devices then read the new metadata as *older* than what
they had and refused it as a rollback attempt. That's TUF doing its job; the fix is
not to fight it manually. It's a good reminder the protection is real, not
theoretical.

---

## E. Yocto / OS / long-term maintenance

**Q: Why Yocto and not just Ubuntu/Debian on the board?**
Control and reproducibility. Yocto lets me pin and rebuild the exact image — kernel,
drivers, runtime, MCU firmware — deterministically, years later, for a security fix.
Qualcomm Linux 2.0 is upstream-first, so I'm not maintaining a private fork forever.
Debian is great for prototyping; Yocto is what you productize on when you owe
someone a decade of updates.

**Q: How does this relate to the Cyber Resilience Act / regulatory stuff?**
Directly. The CRA (EU Cyber Resilience Act) pushes vendors toward maintained,
updatable products with a known software bill of materials. A reproducible Yocto
image (SBOM-able), a signed OTA path (TUF), and separated update lanes are exactly
the ingredients those obligations require. This demo is a small, honest instance of
a compliant update story.

**Q: How do you rebuild the image in two years?**
The Yocto recipes pin the layers and sources; `kas` captures the exact config. Same
inputs, same image. Upstream-first keeps the delta to maintain small. That's the
whole promise — "works on my board today" becomes "I can regenerate this exact
image on demand."

---

## F. AI-assisted development (slide 22)

**Q: How much of this did the AI actually write?**
A lot of the typing — the container glue, the server configuration, iterating
against real hardware. What stayed human was the architecture, the "that's the wrong
approach," and the debugging judgment when something failed in a way the model
couldn't see. The honest framing: the agent compressed the idea-to-running-on-
hardware loop; it didn't replace the engineer's judgment.

**Q: Would you trust it unattended?**
No — and I didn't. It runs against real hardware under direction, and the
interesting bugs were exactly the ones that needed a human to notice the symptom
didn't match the explanation (the silent LED, the all-zero model scores). The value
is velocity with a human in the loop.

---

## G. Business / strategy / "so what"

**Q: Why should I care — this is a colour-LED toy.**
The toy is a stand-in. Replace "recognise a colour word" with "recognise a fault
sound on a machine," and "light an LED" with "flag maintenance," and every piece is
the same: learn from field data, deploy to a fleet over the air, maintain it for
years. The one-hour demo is a compression of a real product lifecycle.

**Q: What does Qualcomm Dragonwing give me that a Raspberry Pi doesn't?**
A production path. The Pi is a lovely prototype; it doesn't give you a
production-class SoC with an NPU, a maintained upstream-first Linux, and a
board-to-board story where the same artifact graduates from bench to product. The
demo's whole point is prototype-to-production on *one* platform.

**Q: Vendor lock-in?**
The ML on-ramp (Edge Impulse) exports standard artifacts; the OS is upstream-first
Yocto; the OTA stack is self-hostable. The lock-in you're choosing is to a
*platform family*, not a proprietary black box, and the escape hatches (standard
model formats, open build system) are real.

**Q: What does this cost to run at scale?**
Self-hosted OTA means no per-device cloud fee — you run the server. The cost is the
engineering to stand up and maintain the pipeline, which is exactly what the
reproducible-build + signed-update discipline is designed to amortise across a large
fleet. I'd size it against the specific fleet rather than quote a number — happy to
take that offline.

---

## H. "I don't know" answers I'll give cleanly

- Exact TOPS utilisation / power draw numbers for a specific workload → *"I'd
  measure that for your model rather than guess."*
- Precise recognition accuracy of today's live-trained model → *"In a hall this
  noisy I don't promise a number; the loop is the point, and for production you'd
  validate against held-out field data."*
- Whether a specific peripheral/sensor is supported on a given board → *"I'd check
  the board's BSP rather than claim it."*
