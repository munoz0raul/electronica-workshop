# Cheat sheet — one page, print it

## The one sentence
> We teach a device two new words with your voices, and ship that new skill to a
> fleet over the air — data to deployed, live, in one hour.

## The arc
Slide 7: "select blue" → **nothing**.  Slide 20: "select blue" → **blue**, "select red" → **red**.
Everything between is *how*, and *why it's production-real*.

## Four pillars
1. One platform, prototype → production (same app/model on UNO Q and VENTUNO Q)
2. The edge learns from real data (you record it; examples-in, model-out)
3. Portability is an architecture (two containers; thin hardware boundary; Yocto)
4. A maintained, secure update path makes it a product (TUF, rollout, two lanes)

## Numbers to have cold
| | UNO Q | VENTUNO Q |
|---|---|---|
| Chipset | QRB2210 | IQ-8275 |
| CPU | quad A53 @2.0 | octa Kryo @2.3 |
| RAM | 2/4 GB LPDDR4 | 16 GB LPDDR5 |
| NPU | — | Hexagon **40 TOPS** |
| MCU | STM32U585 @160 | STM32H5F5 @250 |

- Model: **10 labels**, acts on 6 (`select` + 5 colours). 1 s window @ 16 kHz. **MFE** + **transfer learning**. Exported `.eim`, **AARCH64, CPU** (not NPU).
- App env: `THRESH 0.70`, `DEBOUNCE 0.5`, `SELECT_COOLDOWN 1.0`, `SELECT_SUPPRESS 8.0`. Web UI **:8000**.
- OTA: server on the VENTUNO (systemd), UI **:8080**, device gateway mTLS **:8443**. Group `demo`. Check-in every few seconds. **TUF**-signed targets.

## The LANDMINE answers (most likely hard questions)
- **NPU?** No — CPU. Tiny model; CPU `.eim` runs identically on both boards; NPU build needs delegate libs we don't mount (hangs, all-zero). NPU = headroom for the next model.
- **Data needed?** Little — transfer learning. That's why live retrain fits the hour.
- **Why 2 containers?** Model = data, not code. Retrain = file swap (app lane, seconds), not OS rebuild.
- **App vs OS update?** Two lanes: app = no reboot, seconds; OS = atomic ostree, reboot, auto-rollback. Retrain all day, still ship a kernel CVE.
- **Cloud?** None. On-prem server on the board itself.
- **Malicious update?** TUF signs metadata; rejects unsigned + older (anti-rollback); mTLS.
- **Why Yocto?** Reproducible, pinnable, rebuildable in 2 yrs for a fix; upstream-first; CRA/SBOM story.
- **AI built it?** Typing yes, under direction; architecture + judgment human.

## Panic buttons (live)
- Mic dead → check **capture device** first (UNO Q = ALSA dev 3); then app up / :8000 shows audio.
- Room can't record → use **pre-recorded dataset** (voice-workshop Releases). Loop identical.
- Model bad / training slow → ship the **staged known-good `.eim`** (Fallback A). The *ship* is the message.
- Rollout won't land → check device check-in + group; **do NOT hand-delete targets** (TUF rollback brick). Fallback B: local restart with new `.eim` present (`kill -9`).
- Colour misfires once → repeat phrase once; move on. Don't debug on stage.

## TODO before the talk
- Confirm the two **data-collection QR URLs** (BLUE left / RED right) scan + record.
- Confirm the **public EI project URL** (still `TODO` in voice-workshop).
- Stage the **known-good fallback `.eim`**.

## Repos (on the closing slide)
- github.com/munoz0raul/electronica-voice-workshop — OTA / fleet / Yocto
- github.com/munoz0raul/voice-workshop — app + Edge Impulse
