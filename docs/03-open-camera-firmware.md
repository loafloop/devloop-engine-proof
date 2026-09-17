# 03 — Open replacement firmware for IP cameras (OpenIPC, Thingino, and the dead ones)

Research date: 2026-09-16. Verified from GitHub repos, LICENSE files, issues and
discussions. openipc.org, thingino.com and all retail/price sites were blocked
from the research sandbox, so **prices are unverified recollection** and items
marked *[not fetched]* rest on search snippets.

## Verdict

There are exactly **two living, general-purpose replacement firmwares** worth
building on in September 2026:

| | OpenIPC | Thingino |
|---|---|---|
| Repo | [OpenIPC/firmware](https://github.com/OpenIPC/firmware) | [themactep/thingino-firmware](https://github.com/themactep/thingino-firmware) |
| License | MIT (firmware) | MIT |
| Activity | commits 2026-09-16, nightlies; ~2.2k stars, 3,450 commits, 225 open issues | commits 2026-09-16, releases every few days; ~2.1k stars, 8,541 commits |
| SoCs | HiSilicon Hi35xx and Goke GK7xxx = mature. Ingenic T-series, SigmaStar SSC335/337, Xiongmai XM5xx = MVP. Fullhan, Novatek, MStar = WIP. Rockchip RV1106 exists in tree, status unclear. | Ingenic T10/T20/T21/T23/T30/T31/A1 only. T40/T41 in progress. |
| Cameras | No vetted model list on purpose (vendors swap SoCs within a model line). Wiki has a "popular models" table. Strongest on generic HiSilicon/Goke **PoE bullets** from AliExpress. | ~120 to 150 hardware variants across ~60 consumer brands, mostly **WiFi**: Wyze Cam v2/v3/Pan v1/v2, Atom Cam 2, Xiaofang, Tapo C100 v5, Eufy, Cinnado, Sonoff S2, about 22 Ethernet (non-PoE) models. |
| Streamer | **Majestic: binary-only, Prosperity Public License 3.0.0 (non-commercial), downloaded from an S3 bucket.** Open alternative Divinus (MIT) lacks ONVIF and motion. | prudynt-t (archived) → **Raptor (GPL-3.0)**: RTSP/RTSPS, WebRTC, RTMP, SRT, MJPEG, H.264/H.265, motion (hardware grid or YOLOv5), IR-cut controller, ONVIF Profile T. |
| Vendor blobs | Prebuilt `libmpi.so`, `libisp.so` etc. for every SoC family. | Ingenic `libimp.so` / `libalog.so` blobs on a vendor 3.10.14 / 4.4.94 kernel. |
| Install | UART + TFTP is the primary path; Coupler flashes via stock web UI on HiSilicon/Goke/XM; USB cloner for Ingenic; SPI programmer; defib for unbrick. | **SD-card installers, no soldering** for many models; keeps `combined_backup.bin` for revert. Also USB cloner, UART, SPI. |
| Pick it when | You want **PoE** and are comfortable opening a camera, reading the SoC marking and using a UART adapter. | You want the cheapest working path with true night vision and can accept **WiFi** cameras on an isolated SSID. |

**Neither is fully open.** "Open firmware" today means an open Linux userland
and build system on top of a vendor kernel tree and proprietary ISP/codec
libraries. OpenIPC additionally ships a proprietary streamer by default. This
still achieves the goal in `01-threat-model.md`: the open userland owns the
network stack, so there is no code path to a vendor cloud. But do not describe
it as auditable end to end. The only effort toward blob-free is
[opensensor/open-tx-isp / OpenIMP](https://github.com/opensensor/open-tx-isp)
(GPL-3.0, active Sep 2026), which reimplements Ingenic's ISP driver and reports
"near-OEM daylight parity" on T31. Night/IR parity is not claimed. Not the
default in Thingino yet.

**Biggest practical gap:** no vetted list of currently purchasable **PoE**
cameras exists for either firmware. The safest true-night-vision buy with a
documented no-solder install is still the **Wyze Cam v3 on Thingino**, WiFi
only. For PoE you buy a HiSilicon/Goke/SigmaStar board camera and verify the
chip yourself.

## Everything else, alive or dead

| Project | License | Last activity | Verdict | Notes |
|---|---|---|---|---|
| [OpenIPC/divinus](https://github.com/OpenIPC/divinus) | MIT | 2026-09-10 | Alive | Open streamer for 20+ SoC families. RTSP, JPEG, fMP4, audio, OSD. ONVIF/PTZ/motion still roadmap. |
| [OpenIPC/coupler](https://github.com/OpenIPC/coupler) | MIT | 2026-05-28 | Alive, WIP | Flash OpenIPC through the stock web upgrade page. Hi3516CV/EV, Hi3518, GK7205/7605, XM/Longse. |
| [OpenIPC/defib](https://github.com/OpenIPC/defib) | MIT | active | Alive | Boot-ROM UART unbrick for 120+ SoCs. |
| [gtxaspec/raptor](https://github.com/gtxaspec/raptor) | GPL-3.0 | 2026-09-15 | Alive | Thingino's current streamer. Needs libimp blobs. |
| [gtxaspec/prudynt-t](https://github.com/gtxaspec/prudynt-t) | – | archived 2026-04-16 | Archived | Replaced by Raptor. Still on Thingino stable branch. |
| [opensensor/open-tx-isp](https://github.com/opensensor/open-tx-isp) | GPL-3.0 | 2026-09-10 | Alive | Open ISP driver + OpenIMP. The blob-free future, not the present. |
| [gtxaspec/wz_mini_hacks](https://github.com/gtxaspec/wz_mini_hacks) | no LICENSE file | 2026-06-09 | **Dormant** | README: "not in active development... Thingino is recommended." SD overlay on stock Wyze firmware, **cloud code stays**. |
| [gtxaspec/wz_flash-helper](https://github.com/gtxaspec/wz_flash-helper) | – | archived 2025-01-04 | Archived | Stock↔Thingino switching for Wyze v2/v3. |
| [EliasKotlyar/Xiaomi-Dafang-Hacks](https://github.com/EliasKotlyar/Xiaomi-Dafang-Hacks) | no LICENSE file | 2023-11-05 | **Dead** | 4.3k stars, 77 open issues, ~3 years silent. Dafang itself is not in Thingino's table (Xiaofang T20L is). |
| [samtap/fang-hacks](https://github.com/samtap/fang-hacks) | CC BY-SA 3.0 | 2017-09 | **Dead** | Original Xiaofang SD overlay. |
| [TheCrypt0/yi-hack-v4](https://github.com/TheCrypt0/yi-hack-v4) | GPL-3.0 | 2020-05 | **Dead** | Hi3518e Yi cameras, overlay. |
| [roleoroleo/yi-hack-Allwinner-v2](https://github.com/roleoroleo/yi-hack-Allwinner-v2) | MIT | 2026-09-05 | Alive | ~20 Yi/Kami Allwinner models. **SD overlay next to stock firmware, not a wipe.** Firmware-version locked. |
| [roleoroleo/yi-hack-MStar](https://github.com/roleoroleo/yi-hack-MStar) | GPL-3.0 | 2026-09-12 | Alive, low support | True replacement ("completely overwrite the original firmware"). Maintainer: "I have no time to support the project." |
| [roleoroleo/sonoff-hack](https://github.com/roleoroleo/sonoff-hack) | GPL-3.0 | 2026-09-05 | Alive | Goke Sonoff cams, SD overlay, cloud stays. Newer Sonoff S2/B1P/PT2 (T23N) are natively in Thingino instead. |
| [MuhammedKalkan/Anyka-Camera-Firmware](https://github.com/MuhammedKalkan/Anyka-Camera-Firmware) | MIT | 2025-02 | Semi-dormant | AK3918 lightbulb cameras. RTSP with auto day/night. OpenIPC marks AK3918EV200 "help needed". |
| [bmork/defogger](https://github.com/bmork/defogger) | none | 2020-07 | **Dead** | D-Link DCS-8000LH only. Not a replacement firmware. No open firmware exists for Blink cameras, only cloud-API bridges. |
| TP-Link Tapo rooting ([tapo-firmware/Tapo_C200](https://github.com/tapo-firmware/Tapo_C200), rooting write-up 2025-07 *[not fetched]*) | mixed | 2025 | Research only | No replacement firmware for Realtek C200 v1. But Thingino natively supports **Tapo C100 v5** and has WIP configs for C110/C200/C210 (T23N). |
| Ubiquiti UniFi Protect *[not fetched]* | Proprietary | – | Local-capable, not open | Console can run with a local admin and no UI.com account, exposes RTSP(S). Current cameras need the console. Trust is in Ubiquiti's closed firmware. |
| Reolink / Amcrest / Dahua / Hikvision on a WAN-blocked VLAN *[not fetched]* | Proprietary | – | Local-capable, not open | Native RTSP/ONVIF. Disable P2P/cloud in settings, then firewall. Reolink battery/WiFi-only models lack RTSP. Fallback only; fails the "unable, not configured" test in `01-threat-model.md`. |

## Cameras to buy

True night vision = IR LEDs **and** a mechanical IR-cut filter. Every price
below is an approximate historical range and **must be checked before buying**.

| Model | SoC | Firmware | Night vision | Network | ~Price | Notes |
|---|---|---|---|---|---|---|
| **Wyze Cam v3** | Ingenic T31X or T31AL + GC2053 | Thingino, SD installer, revertible | Yes: IR LEDs + IR-cut, Thingino controls both | WiFi only | $20–36 | The reference cheap pick. **Long-term pass: Wyze's product page says "no longer available and won't be coming back"; firmware still updated. Buy remaining or secondhand stock, plus spares.** Two hardware variants (WiFi chip ATBM6031 vs RTL8189FTV), both supported. USB-Ethernet/PoE-to-USB adapters listed in Thingino accessories for power. |
| Wyze Cam Pan v2 | T31X | Thingino | Yes | WiFi | $40–50 | Pan-tilt. May be discontinued. |
| Wyze Cam Floodlight v1, Doorbell v1/v2 | T31X/T31AL, T30X | Thingino | Yes (doorbell IR-cut unverified) | WiFi | – | |
| **Wyze Cam v4 / Pan v3 / OG** | T41 / T31 secure-boot | **Not supported** | – | – | – | Wyze burns secure-boot e-fuses on newer models. Pan v3 needs a physical SoC swap. "All the newer Ingenic based devices" validate the bootloader signature. **Buy older T31 stock.** |
| Atom Cam 2 (Japan) | T31X GC2053 | Thingino | Yes | WiFi | ¥4,000 | No SD installer image yet. |
| Cinnado D1 (2K/3K) | T23N / T31L + SC2336 | Thingino, SD installer | Yes | WiFi | $20–30 | |
| Galayou G7/Y4, Wansview W7, Jooan A6M/Q3R, Aosu C5L, Aoqee C1, WUUK Y0510, Sonoff S2 | T23N / T31L | Thingino, SD installers | Yes (typical) | WiFi | $20–35 | |
| TP-Link Tapo C100 **v5** | T23N or T31L | Thingino | Yes | WiFi | $20 | Revision-specific. Other revisions are Realtek and unsupported. |
| Eufy E210/E220/C120 | T31X + SC33xx | Thingino | Yes | WiFi | $40–60 | |
| **Imou Ranger 2 IPC-A22E-E** | T31N GC2053 | Thingino | Yes | **Ethernet** + WiFi, no PoE | $30–40 | Pan-tilt. One of the few Ethernet consumer cameras in the list. |
| **Feisda WF-HD620** | T31X JXQ03 | Thingino | Yes (typical) | **Ethernet** + WiFi | $20–30 | Maintainer-named Ethernet model. |
| Jienuo JN-107-AR (D/E) | T31L GC2083 / T31X SC5235 | Thingino | Yes (typical) | Ethernet + WiFi | $20–30 | |
| Wansview W6, XVIM IPCAM-100, Victure PC420, ZTE K540, TPTEK WOR504JCH, Wanjiaan G7, Jooan F2T | T21N/T31X/T30X/T20L | Thingino | Varies | Ethernet + WiFi | – | |
| Vanhua bare modules (Z55/Z55I GC4653, S37I IMX307, H33/L34 GC2083) | T31L/N/X | Thingino | Depends on lens/board kit | Ethernet (PoE via splitter) | $15–30 | For building your own housing. |
| Anjoy MS-J10 / YM-J10D | SigmaStar SSC335/337 + IMX307 | OpenIPC | IR-cut typical, verify | Ethernet, PoE variants exist | $25–40 | IMX307 is a strong low-light sensor. |
| Xiaomi CMSXJ25A | SSC325 GC2053 | OpenIPC | Yes | Ethernet + WiFi | – | |
| Xiaomi MJSXJ03HL | T31N JXQ03 | OpenIPC **and** Thingino | Yes | WiFi | – | |
| **Generic AliExpress Hi3516EV300/EV200 or GK7205V200/V300 + IMX307/IMX335 PoE bullet** | HiSilicon / Goke (mature tier) | OpenIPC via Coupler or UART+TFTP | IR-cut + IR LEDs on most bullets, verify listing | **PoE** | $25–45 | The real PoE route. Pitfalls: some GK7205V300 boards have a password-locked U-Boot (`HI2105CHIP` worked on one, failed on another), NAND boards are problematic, 8 MB flash limits you to OpenIPC Lite. |

## OpenIPC details

- **Finding a supported camera:** open the case, read the SoC marking, look it
  up. On a camera you already own with SSH, run `ipctool`. The wiki is explicit:
  "there cannot be provided a definitive list of compatible devices...
  manufacturers tend to change hardware design and swap components even within
  the same model line"
  ([guide-supported-devices.md](https://github.com/OpenIPC/wiki/blob/master/en/guide-supported-devices.md)).
  Common working combos named in the wiki: HI3518EV200, HI3516EV300, T31X/T31ZX,
  SSC335 with IMX307 / SC2135 / JXQ03 / GC2053.
- **Features (Majestic):** H.264/H.265/MJPEG; RTSP, HLS, HTTP-MJPEG; ONVIF with
  WS-Discovery and mDNS; snapshots; MP4 recording; motion detection with a
  `motion.sh` hook and pre-roll; **night mode = automatic IR-cut switching via
  light sensor, gain thresholds or exposure; PWM IR dimming on HiSilicon/Goke**;
  audio Opus/AAC/G.711; two-way audio; RTMP push; MQTT telemetry in Ultimate
  builds ([majestic-streamer.md](https://github.com/OpenIPC/wiki/blob/master/en/majestic-streamer.md)).
- **Flashing:** UART 3.3 V + U-Boot + TFTP is the documented primary path for
  HiSilicon, Goke, SigmaStar and XM530. None of those guides mention SD-card
  flashing. Coupler does it through the stock web UI for supported families;
  failure means TFTP recovery. Ingenic: USB cloner by shorting flash pins 5–6.
  Unbrick: defib ([installation.md](https://github.com/OpenIPC/wiki/blob/master/en/installation.md)).
- **Pain points:** Majestic source request open since 2022-05-22 (firmware issue
  230) and unanswered. "We always recommend using only Lite firmware with 8M"
  flash. Stable "latest" tag lags nightlies by months. Paid commercial support
  is offered, which explains the proprietary streamer.

## Thingino details

- **Camera list:** [docs/supported_hardware.md](https://github.com/themactep/thingino-firmware/blob/stable/docs/supported_hardware.md)
  and the [wiki Cameras page](https://github.com/themactep/thingino-firmware/wiki/Cameras).
  139 profiles under `configs/cameras` on master. GitHub tree views truncate at
  100 rows, so a naive scrape misses the Wyze entries.
- **Install:** write the SD installer image to a ≥128 MB card, power on, wait
  3–4 minutes for the camera's access point, run `sysupgrade -f`. The installer
  saves `combined_backup.bin`, unique per camera. **Lose it and there is no way
  back to stock.** Revert by renaming it `autoupdate-full.bin`. Some models need
  a flash glitch or disassembly; some are one-way
  ([wltechblog/thingino-installers](https://github.com/wltechblog/thingino-installers),
  [camera-recovery.md](https://github.com/themactep/thingino-firmware/blob/stable/docs/camera-recovery.md)).
- **Branches:** stable "ciao" = prudynt + onvif_simple_server. master = Raptor
  + experimental mainline U-Boot (needs UART and unbrick skills). Use stable.
- **Features:** RTSP + ONVIF, motion detection, **night mode with IR-cut, IR-LED
  and white-light control**, PIR, two-way audio, web UI, MQTT control topics
  for motor, snapshot, record, motion, IR/white light, day/night, privacy
  ([mqtt-subscriptions.md](https://github.com/themactep/thingino-firmware/blob/stable/docs/mqtt-subscriptions.md)).
- **Ethernet/PoE reality:** the maintainer in discussion 242 (Oct 2024) named
  only Feisda WF-HD620 and "all Vanhua modules" as Ethernet. The table now flags
  ~22 Ethernet variants. **None advertise 802.3af PoE**; budget a PoE splitter.
- Secure boot: Wyze Pan v3, Cam v4 and Roku variants validate the bootloader
  signature (discussions 470, 764). Expect this to spread across new Ingenic
  consumer cameras. Buy known-older stock or verify the exact revision.
  **Long-term pass:** Tapo C500 (T23) units with a bootloader dated 2026-01-06
  no longer offer the autoboot prompt (Thingino issue 1641, open). The
  lock-down is reaching T23. See `07-long-term.md`, section 4.

## Not verified this round

- openipc.org supported-hardware pages, thingino.com camera pages (blocked).
- All prices.
- Wyze Cam v3 IR spec page (850/940 nm, IR-cut) — from Wyze marketing, not
  fetched.
- Whether Thingino publishes motion events over MQTT (control topics verified,
  event publishing not).
- ipcamtalk threads on "cameras with open firmware and Ethernet" (blocked); they
  are the best community source for PoE picks and worth reading manually.
