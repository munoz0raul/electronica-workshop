# Architecture — from mic to fleet

Grounded in [electronica-voice-workshop `docs/ARCHITECTURE.md`](https://github.com/munoz0raul/electronica-voice-workshop)
and the voice-workshop app. This is the "how it actually works" layer behind
slides 12–21.

## The data path, end to end

```
microphone ──ALSA──▶ app container ──Edge Impulse .eim──▶ "select blue" (conf ≥ 0.70)
                          │
                          ├──▶ web UI on :8000 (shows what it heard)
                          └──▶ serial /dev/mcu-console ──MCUmgr SMP grp 64──▶ Zephyr MCU ──▶ LEDs
```

## The two containers (slide 12)

| Container | Job |
|---|---|
| `webapp-led-mcu-voice` | **The app.** Captures mic, runs the `.eim`, applies the grammar/debounce, drives the MCU over serial, serves the web UI on port **8000**. |
| `webapp-led-mcu-voice-assets` | **Model delivery.** Puts `deployment.eim` into a shared volume at `/var/local/assets/` and signals the app to reload when the model changes. |

They share a volume. **Why the split is the whole point:** the model is *data*, not
code. A retrain changes only the file the assets container delivers — so shipping a
new model is an **app-level update** (swap a file, no reboot), never an OS rebuild.
That decoupling is what makes "retrain ten times a day" realistic.

Key app constants (voice-workshop): model at `/app/deployment.eim`, MCU port
defaults to `/dev/mcu-console`, the label/colour sets, and the threshold/debounce
env from [04-edge-impulse.md](04-edge-impulse.md). On the UNO Q the mic is ALSA
device 3 (`PA_ALSA_DEVICE=3`) — a concrete example of the hardware boundary.

## The hardware boundary (slide 13)

What travels unchanged across boards: the `.eim`, the Python app, the container
images, the web UI. What's board-specific and lives at the boundary: **which ALSA
device is the mic**, **which serial port reaches the MCU**. Keep that boundary thin
and declared in config, and the same artifact runs everywhere. The mic is the most
visible instance — the demo literally depends on selecting the right capture device.

## Restarting the app on a new model — the gotcha

When a new `.eim` lands, the app must reload it. The assets container restarts the
voice app, and it has to use **`kill -9` (SIGKILL)** — a plain SIGTERM is dropped
by the init process inside the voice app's PID namespace, so a graceful stop never
arrives and the model wouldn't reload. Good concrete example that "reload the
model" is more than `cp`.

## Yocto / the OS (slide 14)

- The OS is built with **Yocto** (Qualcomm Linux 2.0, upstream-first, `meta-qcom`
  overlays). The image — kernel, drivers, container runtime, the MCU firmware that
  gets loaded at boot — is **defined, pinned and reproducible**.
- Images are **ostree**-based: atomic, versioned filesystem commits. Each board has
  its own ostree repo / commit (UNO Q, VENTUNO Q, IQ-9075 EVK).
- This is the answer to "can you rebuild this in two years for a CVE fix?" — yes,
  reproducibly, because the recipe pins everything. Upstream-first means not
  carrying a fork forever. Ties to **CRA** (EU Cyber Resilience Act): a maintained,
  rebuildable, SBOM-able image is what long-term obligations require.

## The OTA / fleet system (slides 16–19)

Self-hosted **Foundries.io**-style update stack. The headline: **it all runs
on-prem, on the VENTUNO Q itself** — no cloud dependency.

- **Update server** runs as a **systemd service** on the VENTUNO Q (not a compose
  app). Web UI + REST on **:8080**; device gateway (mutual-TLS) on **:8443**; state
  under `/var/local/datadir`.
- On each device: **aktualizr-lite** pulls targets; **fioconfig** applies config;
  devices check in on a short interval (seconds).
- **A "target"** = a signed, versioned description of the complete software a device
  should run: the ostree commit (the OS) **plus** the staged compose apps. A
  **rollout** offers a target to a **device group** (e.g. `demo`).
- **Push-to-deploy:** a git commit of the new model is picked up by an automatic
  build (a poll-and-build timer, ~2-minute cadence), which produces a new target.
  I create a rollout; devices pick it up on their next check-in.
- **TUF** (The Update Framework) signs all the metadata. A device rejects anything
  unsigned, mis-signed, or older than what it runs (anti-rollback).

### App update vs OS update (slide 21) — the two lanes

| | App update | OS update |
|---|---|---|
| What changes | a compose app / the model file | the ostree OS commit |
| Reboot? | no | yes (boot into new deployment) |
| Speed | seconds | minutes |
| Rollback | redeploy previous app | atomic — falls back if new image won't boot |
| In the demo | the live "select blue" ship | the separate, slower track we *don't* run live |

Decoupling the lanes is the maintainability payoff: retrain endlessly on the app
lane without ever disturbing the base OS, yet still deliver a kernel/security fix on
the OS lane when needed.

### Fleet view (slide 19)
The server UI lists every device and the target it's running. I watch the new
target propagate offered → installed across the `demo` group — the "verify" in
"deploy and verify."

## Operational landmines I learned building this (keep for Q&A, not slides)

- **CPU `.eim`, not QNN.** The NPU-accelerated model needs Qualcomm delegate
  libraries mounted into the container; absent them it hangs and scores all-zero.
  The CPU build has no such dependency. (See [04-edge-impulse.md](04-edge-impulse.md).)
- **`kill -9` to reload the model** — SIGTERM is swallowed by the app's PID-ns init.
- **Don't delete targets in the server UI** — it can regress the generated TUF
  version and a device then reads the new metadata as *older* than what it has and
  refuses it ("rollback attempt"). Recovery is clearing the device's cached
  metadata. Lesson: TUF's anti-rollback is doing its job; don't fight it by hand.
- **Keep unrelated heavy apps out of a board's target** — bundling an extra app
  that needs libraries missing on a read-only rootfs made an app *install fail*, not
  just run slow. Each board's target should carry only what that board can mount.
- **MCU firmware is part of the OS image**, loaded at boot from the Linux rootfs —
  so a firmware fix (like the VENTUNO RGB `set_leds` command) is an OS rebuild, not
  an app update. Know which lane a given fix lives in.

These are not failure stories to volunteer on stage — they're the proof, if a
sceptic pushes, that this is a real system with real edges, not a happy-path demo.
