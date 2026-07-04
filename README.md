# mp-free

**An open-source hi-fi network music player.** Bit-perfect FLAC playback up to 24-bit / 192 kHz, managed from your phone over WiFi. Fully FOSS — hardware, firmware, and app.

> ⚠️ **Status: pre-alpha.** This project is in early bring-up. Expect things to be incomplete, to change, and to break. Follow along or jump in.

---

## What it is

mp-free is a from-scratch, fully open hi-fi music player built around a clean two-chip architecture:

- **Lossless audio, done right** — FLAC up to 24-bit / 192 kHz, decoded on a dedicated real-time audio MCU and streamed to a PCM5102A I²S DAC.
- **Networked storage** — your music library lives on an SD card in the player. Browse, upload, download, and delete files from your phone over WiFi.
- **Glitch-free by design** — the SD card is owned solely by the audio MCU, so the audio path is fully isolated from network activity. Uploading a file or browsing the library never interrupts playback.
- **Live telemetry** — RAM usage, SD card usage, and WiFi signal strength, viewable from the phone app.

## Architecture

```
  Phone app
     │  WiFi
     ▼
  ESP32-C3            ── WiFi + FTP/web server, network bridge
     │  SPI (+ ready line)
     ▼
  STM32              ── real-time audio processor; sole owner of the SD card
     ├── SDMMC ──▶  SD card        (FLAC library)
     └── SAI/I²S ─▶ PCM5102A DAC ─▶ line out
```

The ESP32 never touches the SD card directly. It relays file operations (list, open, read, write, delete) to the STM32 over SPI; the STM32 runs them against FatFs behind a mutex, always servicing the audio DMA first. This keeps playback bulletproof regardless of network load.

**DAC clocking note:** the PCM5102A uses its internal PLL (SCK tied to ground), so the player feeds it only BCK / LRCK / DIN — no external master clock required.

## Hardware

| Role | Prototype part | Target (custom PCB) |
|------|----------------|---------------------|
| Audio processor | STM32H7 (dev board) | STM32F446 / STM32H5 (LQFP48–64) — under evaluation |
| DAC | GY-PCM5102A module | PCM5102A |
| Network bridge | ESP32-S3 (dev board) | ESP32-C3 |
| Storage | microSD (4-bit SDMMC) | microSD |

Schematics and PCB design files will live under [`/hardware`](hardware/) once the breadboard prototype is verified.

## Roadmap

Each milestone is only built on a *proven* layer beneath it.

- [ ] **1.** Sine table → SAI/DMA → PCM5102A (clean tone, no SD/FLAC/WiFi)
- [ ] **2.** WAV from SD (SDMMC + FatFs + double-buffered I²S)
- [ ] **3.** FLAC decode
- [ ] **4.** ESP32: WiFi + FTP/web server (standalone, local storage)
- [ ] **5.** SPI link + file-operation RPC proxy (integration)
- [ ] **6.** Phone app + telemetry
- [ ] **7.** Custom PCB

## Repository layout (planned)

```
firmware-stm32/   real-time audio: SDMMC, FatFs, FLAC decode, SAI/I²S, SPI slave
firmware-esp32/   WiFi, FTP/web server, SPI-to-filesystem bridge
app/              phone app: file management + telemetry dashboard
hardware/         schematics and PCB design files
docs/             design notes, protocol spec, build guide
```

## Building

To be documented as each subsystem comes online. See the roadmap for current state.

## Contributing

Contributions, issues, and ideas are welcome — especially once bring-up stabilizes past milestone 3. If you're building along on the same hardware, bug reports and design feedback are gold at this stage.

## License

mp-free uses a per-domain license stack so that hardware, software, and documentation are each covered appropriately:

| Component | License |
|-----------|---------|
| Firmware & software (`firmware-*`, `app/`) | **GPL-3.0-or-later** |
| Hardware design files (`hardware/`) | **CERN-OHL-S v2** |
| Documentation (`docs/`, this README) | **CC-BY-SA-4.0** |

This is a strongly reciprocal ("copyleft") stack: derivatives must remain open. If you prefer maximum adoption over guaranteed openness, a permissive alternative would be Apache-2.0 / CERN-OHL-P / CC-BY-4.0.

Full license texts belong in the repo: add `LICENSE` (GPL-3.0), `LICENSE.hardware` (CERN-OHL-S), and note the docs license in `docs/`. Canonical texts are available from the GNU Project, the CERN OHL repository, and Creative Commons respectively.

> Not legal advice. For commercial use or anything with real liability exposure, consult a lawyer.
