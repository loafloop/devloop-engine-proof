# Open, cloud-free CCTV: research and build notes

> **Branch note.** This repository used to hold a throwaway test page for a dead
> tooling experiment. That content was removed on this branch, which is a
> temporary home for research into a fully local CCTV system built from open
> firmware and open hardware. First pass 2026-09-16; long-term pass 2026-09-17.

## The questions, answered

**Does ESP32 + camera have night vision?** No, not as sold. The AI-Thinker
ESP32-CAM and every mainstream ESP32-S3 camera board has a fixed IR-cut lens,
no IR illuminator, and a white (not IR) flash LED. Night vision means buying an
"850nm" no-IR-filter OV2640 module, adding your own 850 nm LEDs, and losing
daytime colour unless you use the one board with a switchable IR-cut filter
(LilyGO T-CameraPlus-S3). On top of that, every ESP32/S3 firmware is MJPEG-only,
so your NVR spends about 0.4 Pi-4 cores per camera transcoding. Verdict: a
cheap auxiliary camera, not the backbone. Details in
[docs/02-esp32-cameras.md](docs/02-esp32-cameras.md).

**Is an "open" stack possible?** Yes, with one honest caveat. Two living
replacement firmwares wipe the manufacturer's software from off-the-shelf
cameras: **OpenIPC** (HiSilicon/Goke PoE cameras, MIT, but its default streamer
is a proprietary binary) and **Thingino** (Ingenic cameras like the Wyze Cam v3,
MIT, GPL streamer). Both still run vendor ISP blobs under an open userland. The
open userland owns the network stack, so there is no code path to a vendor
cloud, which is the property that matters. But it is not auditable end to end.
Details in [docs/03-open-camera-firmware.md](docs/03-open-camera-firmware.md).

**The NVR side is fully open and mature.** Frigate (MIT, 0.18, Sep 2026) on an
Intel N100 mini PC with OpenVINO detection needs no add-on accelerator and no
cloud. Three default outbound calls must be switched off; they are listed with
config in [docs/04-nvr-software.md](docs/04-nvr-software.md).

## The recommended stack

| Layer | Pick | Why |
|---|---|---|
| **Wired cameras with night vision** | Generic **SigmaStar SSC338Q/SSC30KQ** (preferred for supply) or Hi3516EV300 / GK7205V300 + IMX307 **PoE bullet** from AliExpress, reflashed to **OpenIPC** | Only route to PoE + open firmware + real IR-cut and IR LEDs. Needs a UART adapter and reading the SoC marking before buying. SigmaStar: not sanctioned, 37% market share, ships OpenIPC from the factory in FPV products; OpenIPC tier is "MVP", verify the exact SoC. |
| **Cheap indoor / WiFi cameras** | **Wyze Cam v3** (T31 revision, not v4/Pan v3) on **Thingino** | $20–36, SD-card install, no soldering, IR-cut + IR LEDs controlled by open firmware. WiFi only, so isolated SSID. |
| **DIY wired node** | **Luckfox Pico Ultra** (RV1106, PoE) on OpenIPC, or **Pi 4 / Zero 2 W** + Arducam IR-cut module on MediaMTX | Luckfox: hardware H.264, PoE, OpenIPC support merged 2026-09-07, but no IR module sold yet. Pi: best docs, but costs more than a commercial camera and **Pi 5 has no hardware H.264 encoder**. |
| **Spyware-proof by construction** | **USB UVC camera with IR** (ELP) on the NVR or a Pi | No CPU, no network stack, cannot phone home. Limited by USB cable length. |
| **NVR** | **Frigate 0.18** on an Intel N100/N150 mini PC (or used i5 Tiny), 16 GB RAM, surveillance HDD | OpenVINO on the iGPU, VAAPI decode, 4–8 cameras comfortably, MIT. |
| **Streaming glue** | go2rtc (bundled) and MediaMTX | Both MIT; MediaMTX phones home nowhere by default. |
| **Network** | Camera VLAN with **no route to WAN**, PoE switch, NVR-only ingress, local NTP, UPnP off | Configs for OpenWrt and OPNsense in [docs/06-network-isolation.md](docs/06-network-isolation.md). |
| **Remote viewing** | **WireGuard on your own router** (or Headscale if you want Tailscale's UX) | Tailscale hosted puts its control plane in the path; Cloudflare Tunnel sees plaintext. Neither is "no cloud". |

**Do not buy:** Google Coral (upstream archived, de-recommended by Frigate),
Wyze Cam v4 / Pan v3 / OG (secure boot, not flashable), Raspberry Pi 5 as a
camera node, anything advertised with "view from anywhere via QR code".

## What is better long term (5 to 10 years)

Second research pass, full detail in [docs/07-long-term.md](docs/07-long-term.md).
The two things you want, auditable firmware and 10-year supply, do not live
on the same camera today. So the long-term design is to **separate the durable
core from the consumable edge**:

- **Durable core, build it first:** Frigate on an Intel-iGPU x86 box, camera
  VLAN with no WAN route, WireGuard on your own router. Best governance in the
  stack (Frigate, Inc., three people with merge rights), OpenVINO on Intel has
  eleven years of backward support with no vendor kernel, recordings are plain
  MP4, and RTSP + ONVIF Profile T will outlive every camera. One Frigate
  upgrade a year. Keep MediaMTX in reserve: go2rtc is one person and has had no
  code change since March 2026. Archive images and model files locally.
- **Consumable edge, 3-to-5-year cycle, chosen by exit cost:**

  | Rank | Route | Horizon |
  |---|---|---|
  | 1 | **Pi 4 / CM4 + switchable IR-cut module + MediaMTX** for the positions that matter. Only DIY route with no blob you cannot rebuild (libcamera's Pi ISP algorithms are BSD-2). In production until **January 2034**. Most labour, ~€145 and 4 W per node. | 10 years |
  | 2 | **OpenIPC on SigmaStar PoE bullets** (Goke/HiSilicon if that is what you can get) for the rest. Frozen 4.9 kernel and blobs, tolerable on a no-WAN VLAN. SigmaStar over HiSilicon on supply grounds. Mirror the release assets: Majestic is a binary fetched from S3 without a hash. | 5 years per model |
  | 3 | **USB UVC IR camera** on the NVR for the one or two positions within 5 m. Nothing to patch, nothing to exit from. | 10 years |
  | 4 | **Thingino on Wyze Cam v3** for cheap indoor cameras. Best firmware, closing hardware supply (v4 is fused, Wyze no longer sells v3). Buy spares now. | 3 to 5 years |
  | 5 | **Luckfox Pico Ultra** as a tinkering node. Vendor kernel 5.10 (LTS ends Dec 2026), 4-to-5-year Rockchip chip cycle, no IR-cut module. | 5 years |
  | 6 | **ESP32-P4.** Hardware encoder is now open source and PoE boards exist, but the only active firmware is a one-person WiFi-only beta. | Re-evaluate 2028 |
  | 7 | **Axis / Hanwha closed cameras** on the VLAN, only if five years of vendor patches after discontinuation matters more than auditability. Three times the price, lowest labour. | 10 years, closed |

- **The 10-year bill for six cameras** (labour excluded, €0.30/kWh, estimates):
  Thingino WiFi ≈ €870, Luckfox ≈ €1,160, USB ≈ €1,250, OpenIPC bullets ≈
  €1,330, Pi 4 nodes ≈ €2,130, Axis ≈ €4,120. At €30/h of your time, Pi and
  Axis converge and OpenIPC bullets stay cheapest.
- **Regulation does not stop you anywhere**, but it reshapes the shelf: FCC
  ban on new Hikvision/Dahua imports effective 2026-07-16; EU Cyber Resilience
  Act makes security cameras "important products" from 2027-12-11 with a
  5-year update duty that pushes vendors toward auto-update mechanisms; wired
  PoE-only cameras sit outside the EU RED radio rules.

## What "no cloud" has to mean

Five behaviours a device must be *unable* to perform, not merely configured
not to: outbound telemetry, cloud relay / P2P hole punching, auto-joining
foreign WiFi, silent firmware updates, and DNS/NTP side channels. Plus a
testable definition of done (24 h packet capture on the camera VLAN shows
nothing but NVR and NTP traffic). See
[docs/01-threat-model.md](docs/01-threat-model.md).

## Documents

| File | Contents |
|---|---|
| [docs/01-threat-model.md](docs/01-threat-model.md) | What to eliminate, trust boundaries, residual risk, definition of done |
| [docs/02-esp32-cameras.md](docs/02-esp32-cameras.md) | ESP32/S3/P4 night vision, 15 firmware projects with alive/dead status, MJPEG cost |
| [docs/03-open-camera-firmware.md](docs/03-open-camera-firmware.md) | OpenIPC vs Thingino, 20 related projects alive/dead, cameras-to-buy table, blob reality |
| [docs/04-nvr-software.md](docs/04-nvr-software.md) | 18 NVR/streaming projects, exact licenses, what phones home, detection hardware, three reference stacks |
| [docs/05-diy-camera-hardware.md](docs/05-diy-camera-hardware.md) | Raspberry Pi (Pi 5 no HW encode), Luckfox RV1106, Milk-V, Sipeed, USB IR cameras |
| [docs/06-network-isolation.md](docs/06-network-isolation.md) | VLAN pattern, OpenWrt/OPNsense/UniFi rules, WireGuard/Headscale/NetBird, leak tests |
| [docs/08-esp32-starter.md](docs/08-esp32-starter.md) | Shopping list for a first ESP32 camera (wired PoE kit or cheapest WiFi kit), what to skip, firmware, first steps |
| [docs/07-long-term.md](docs/07-long-term.md) | **Second pass.** Production-until dates, regulation, standards, firmware and NVR governance with measured bus factors, detection hardware longevity, DIY routes with a 10-year cost table, ranked verdict |

## Project status index (alive / dead, as of 2026-09-16)

**Alive and recommended:** Frigate, OpenIPC, Thingino, go2rtc, MediaMTX,
Viseron, ZoneMinder, Moonfire NVR, s60sc/ESP32-CAM_MJPEG2SD, ESPHome
esp32_camera, Tasmota32 webcam, rzeldent/esp32cam-rtsp, Headscale, NetBird.

**Alive but caveated:** go2rtc (one person, no code change since Mar 2026, fine
while pinned inside Frigate), OpenIPC Majestic (proprietary binary), Shinobi (non-OSI
licence), Scrypted (NVR plugin paid/closed), Kerberos Agent (phones home unless
`AGENT_OFFLINE=true`), yi-hack-Allwinner-v2 and sonoff-hack (SD overlays that
leave the vendor cloud firmware in place), Raptor/prudynt (need Ingenic blobs),
r4d10n/esp32p4-uvc-video and ESP32CAM-ONVIF (P4, hobby-grade), OpenNVR (too
young).

**Dead or dormant:** Xiaomi-Dafang-Hacks (2023), fang-hacks (2017), yi-hack-v4
(2020), defogger (2020), wz_mini_hacks (dormant, points to Thingino), ShinobiCE
(2021), motionEyeOS (2020), easytarget/esp32-cam-webserver (EOL 2024),
arkhipenko/esp32-cam-mjpeg-multiclient (2020), geeksville/Micro-RTSP (2023),
ArduCAM ESP32S UNO (2021), Google Coral upstream (archived 2025–2026).

## About the LG TV premise

What was verifiable this round: around 2026-09-11 Tom's Guide, citing Ars
Technica's coverage of a Gamers Nexus and Level1Techs investigation, reported
that some LG webOS TVs **scan the local network for other devices** such as
phones and watches, and LG confirmed the TVs "feature the ability to scan for
and connect to nearby devices on the same network", calling it standard smart
TV behaviour. That concerns a network the TV has already joined. The specific
claim that LG TVs **auto-join open or previously-known WiFi access points** was
**not** found in any news report, LG statement or documented test; the
relevant forums were unreachable from the research sandbox, so it stays
unverified. The verified behaviour alone justifies the same design decision:
cameras with WiFi radios and vendor firmware do not belong on your network.

## Research limits

Eight parallel researchers across two passes worked from GitHub repos, LICENSE files, release
feeds, issues and official docs-as-source. Vendor sites (openipc.org,
thingino.com, raspberrypi.com, arducam.com, espressif.com, coral.ai, all
retailers) and most forums were egress-blocked, so **every price is an
unverified recollection**, and items marked *(snippet only)* or *[S]* rest on
search-result text. Each doc ends with a "not verified" list. Read those before
spending money.
