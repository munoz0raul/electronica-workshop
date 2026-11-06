# Glossary — every term in the deck, one line

Terms and acronyms a question might turn on. One line each; depth lives in the
other docs.

## ML / audio
- **Keyword spotting** — classifying short audio windows as one of a fixed set of words (our 10 labels).
- **MFE (Mel Filterbank Energies)** — signal-processing block that turns a 1 s clip into a Mel-scale spectrogram ("a picture of the sound").
- **MFCC** — MFE plus a cepstral (DCT) transform; classic speech feature, usually unnecessary for small keyword nets.
- **Transfer learning** — fine-tuning a net already trained on a large corpus on our 10 labels; why training takes minutes.
- **Window / stride** — 1000 ms analysis window at 16 kHz; the chunk the model classifies.
- **Negative classes** — `noise`, `unknown`, `update`; teach the model what *isn't* a command.
- **Confidence threshold** — min score (0.70) before a detection counts.
- **Debounce / cooldown / suppress** — timing guards (0.5 / 1.0 / 8.0 s) so one utterance is one action.
- **`.eim`** — Edge Impulse's self-contained Linux model executable; built here for AARCH64, CPU.
- **Edge Impulse** — the tooling we use to collect data, build the impulse, train, and export.

## Hardware
- **UNO Q** — prototyping board: Dragonwing QRB2210, quad Cortex-A53, STM32U585 MCU.
- **VENTUNO Q** — production-class board: Dragonwing IQ-8275, octa-core Kryo, 16 GB LPDDR5, 40-TOPS Hexagon NPU, STM32H5F5 MCU.
- **IQ-9075 EVK** — higher-end evaluation kit; the "fourth platform" in the build.
- **Dual-brain** — a Linux applications SoC plus a real-time Cortex-M33 MCU on one board.
- **Dragonwing** — Qualcomm's brand for the intelligent-edge / IoT platform family.
- **NPU (Neural Processing Unit)** — the Hexagon tensor accelerator; 40 dense TOPS on the VENTUNO. Not used by this model.
- **TOPS (Tera Operations Per Second)** — throughput measure for the NPU.
- **Hexagon** — Qualcomm's DSP/NPU architecture.
- **Kryo** — Qualcomm's CPU core brand.
- **MCU (Microcontroller Unit)** — the STM32 Cortex-M33 driving the LEDs in real time.
- **Zephyr** — the real-time OS running on the MCU.
- **MCUmgr / SMP** — the management protocol (custom group 64) the Linux app uses over serial to command the MCU.
- **LPUART** — low-power UART; the serial link to the MCU (115200 8N1, `/dev/mcu-console`).
- **ALSA** — Linux audio layer; the mic is an ALSA capture device (device 3 on the UNO Q).

## OS / build
- **Yocto** — the embedded-Linux build system that defines and pins the image.
- **Qualcomm Linux 2.0 (QLI)** — the upstream-first Yocto-based Linux foundation for the platform.
- **`meta-qcom`** — the Yocto layer carrying Qualcomm board support.
- **kas** — the tool that captures/pins the Yocto build config.
- **ostree** — atomic, versioned filesystem commits; how the OS image updates and rolls back.
- **SBOM (Software Bill of Materials)** — the inventory of everything in the image; needed for maintenance/compliance.
- **CRA (Cyber Resilience Act)** — EU regulation pushing maintained, updatable, SBOM-able products.

## Deployment / OTA
- **OTA (Over the Air)** — updating software remotely, no cable.
- **aktualizr-lite** — on-device agent that pulls targets from the server.
- **fioconfig** — on-device agent that applies configuration.
- **Target** — a signed, versioned description of the complete software a device should run (ostree commit + apps).
- **Rollout** — offering a target to a device group.
- **Device group** — a named set of devices (e.g. `demo`) a rollout addresses.
- **TUF (The Update Framework)** — signs update metadata; enforces authenticity and anti-rollback.
- **mutual TLS (mTLS)** — device and server each authenticate the other (device gateway on :8443).
- **poll-and-build** — the timer that turns a git commit into a new target automatically.
- **Foundries.io** — the lineage of the self-hosted OTA stack used here.

## App
- **`webapp-led-mcu-voice`** — the app container: mic → model → LEDs + web UI on :8000.
- **`webapp-led-mcu-voice-assets`** — the sidecar that delivers `deployment.eim` and triggers a reload.
- **Shared volume** — where the model file lands (`/var/local/assets/`) so the app can pick it up.
- **`select <colour>`** — the two-word command grammar the app acts on.
