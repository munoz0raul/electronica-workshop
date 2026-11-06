# Hardware — the two brains

The one idea to own: **both boards are dual-brain.** A Linux applications
processor (big cores, runs the OS, the AI, the web app) *and* a real-time
microcontroller (Cortex-M33, drives the LEDs deterministically) on the same board,
talking over a serial link. The Linux side decides *what* colour; the MCU side
makes the LEDs do it with hard timing.

## Spec comparison (from deck slide 6)

| | **UNO Q** (prototyping) | **VENTUNO Q** (production-class) |
|---|---|---|
| Chipset | Dragonwing **QRB2210** (Q2) | Dragonwing **IQ-8275** (IQ8) |
| Applications CPU | Quad-core Cortex-A53 @ 2.0 GHz | Octa-core Qualcomm Kryo @ 2.3 GHz |
| RAM | 2 / 4 GB LPDDR4 | 2× 8 GB LPDDR5 (16 GB) |
| Storage | 16 / 32 GB eMMC | 64 GB eMMC |
| NPU | — (CPU/GPU only) | Hexagon Tensor, **40 dense TOPS** |
| Real-time MCU | **STM32U585**, Cortex-M33 @ 160 MHz | **STM32H5F5**, Cortex-M33 @ 250 MHz |
| Connectivity | Wi-Fi 5, BT 5.1 | Wi-Fi 6, BT 5.3, 2.5 GbE |
| OS | Ubuntu/Debian or Yocto Linux | Debian or Yocto Linux |

A third board, the **IQ-9075 EVK**, also appears in the fleet build (its own OS
image / ostree); it's the higher-end evaluation kit and shows the *same* app
running on yet another platform. Mention it only if asked "how many platforms" —
the deck's headline is "four platforms, three in the live rollout."

> **MCU part-number check (I verified this during the build):** UNO Q =
> **STM32U585**, VENTUNO Q = **STM32H5F5**. The deck slide and my firmware work
> agree. If a slide elsewhere swaps them, the firmware repo is authoritative
> (the VENTUNO's Zephyr firmware is built for the H5).

## Why dual-brain matters (the talk-track reason)

LEDs — especially an 8×13 matrix plus an RGB indicator — need deterministic
timing. You don't want a Linux scheduler hiccup to glitch the display. So:

- **Linux side** (the Kryo/A53): captures the mic, runs the Edge Impulse model,
  decides "the user said *blue*," serves the web UI.
- **MCU side** (the Cortex-M33 running Zephyr): receives a compact command over
  serial and renders the LEDs with real-time guarantees.

This is a microcosm of how real industrial/robotics products are built: a rich OS
for intelligence and connectivity, a microcontroller for the motion/IO that must
never miss a deadline. Same pattern, from a doorbell to a robot arm.

## The link between the brains (if someone goes deep)

- The Linux app talks to the MCU over a **serial port** (`/dev/mcu-console`, an
  LPUART, 115200 8N1).
- The protocol is **MCUmgr / SMP** with a custom command group (group 64). The
  app sends a "render these LEDs" command; the Zephyr firmware executes it.
- The colour fix on the VENTUNO was exactly this: a `set_leds` command the baked
  firmware didn't have yet, so the RGB LED stayed dark until the firmware was
  rebuilt. Good concrete story about *why the MCU firmware is part of the OS image*
  (it's loaded at boot from the Linux rootfs), not part of the app update loop.

## Where the model actually runs — pre-empt this

The VENTUNO's headline number is the **40-TOPS Hexagon NPU**. The keyword model
does **not** use it — it runs on the **CPU**. That's deliberate and defensible:

- A 10-keyword, 1-second-window spotter is tiny. The CPU does it in real time with
  margin; the NPU would be overkill.
- The portable `.eim` is built as a CPU target so the *identical* file runs on the
  UNO Q (which has no NPU) and the VENTUNO. Portability beats peak throughput here.
- The NPU is the platform's **headroom**: it's there for when you grow from
  keyword spotting to, say, a vision model or a larger audio model. It's the reason
  you'd *productize* on the VENTUNO even though the prototype ran fine on the UNO Q.

Full reasoning (including the practical "the CPU build just runs, the NPU build
needs delegate libraries we don't ship") is in [07-anticipated-qa.md](07-anticipated-qa.md).
