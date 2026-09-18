# 09 — Open camera platforms that are not toys: options, pros, cons

Written 2026-09-18 from the two research passes (docs 02 to 07). "Not a
toy" means: real sensor with IR-cut and IR illumination available,
weatherproofing available, wired Ethernet or PoE, hardware H.264/H.265,
RTSP, and maintained software.

Three layers can be open or closed on any camera: the bootloader and kernel,
the ISP and video encoder, and the userland streamer. Only one platform below
is open on all three, and it pays with CPU encode.

| Rank | Platform | Bootloader / kernel | ISP / encoder | Streamer | Night vision | Wired / PoE | ~Cost per camera | Horizon |
|---|---|---|---|---|---|---|---|---|
| 1 | **OpenIPC on SigmaStar / HiSilicon / Goke PoE bullet** | Open U-Boot; vendor 4.9 kernel, unpatched (HiSilicon has an experimental Linux 7.0 path) | Vendor blobs | **Majestic: proprietary binary** (Divinus open but no ONVIF/motion) | Built in: IR-cut + IR LEDs | **PoE** | $25–45 | 5 y per model |
| 2 | **Thingino on Ingenic T31** (Wyze v3, Cinnado D1, Imou Ranger 2) | Open U-Boot; vendor 3.10 kernel, unpatched; experimental 6.11/7.1 configs | libimp blobs; open-tx-isp/OpenIMP in progress | **Open (GPL)** | Built in | WiFi mostly; ~22 Ethernet models, no PoE | $15–40 | 3–5 y, supply closing |
| 3 | **Raspberry Pi 4 / CM4 + IR-cut module + MediaMTX** | Pi kernel fork (ISP and encoder drivers not mainline); closed VideoCore firmware in the path | **Open 3A algorithms** (libcamera BSD-2); encoder in VideoCore firmware | **Open (MIT)** | Third-party IR-cut boards; add illuminator | PoE HAT | ~€145 | **10 y** (production to Jan 2034) |
| 3b | **Raspberry Pi 5 + Camera Module 3 / IR-cut module + MediaMTX** | Mainline (`rp1-cfe`, `pisp_be`) | **Open**, software H.264 (~0.5–1 core per 1080p25) | Open | Same | PoE HAT (third party) | ~€170 | 10 y (production to Jan 2036) |
| 4 | **Luckfox Pico Ultra (RV1106)** with OpenIPC or stock SDK | Vendor 5.10.160 (LTS EOL 2026-12-31); mainline boot-level arriving | rockit/rkaiq/mpp blobs | Majestic (OpenIPC) or stock rkipc | **None sold**; DIY IR-cut coil + LEDs | **PoE** | ~$30 board + housing | 5 y (Rockchip chip cycle) |
| 5 | **USB UVC IR camera on a Linux host** | n/a: no CPU | n/a | n/a: host does everything | Built in: IR-cut + IR LEDs | USB, ~5 m; no PoE | $35–70 | 10 y, near the NVR only |
| 6 | **Seeed reCamera (SG2002)** | Open hardware (KiCad), open SDK, mainline RISC-V work | Sophgo middleware blobs | DIY RTSP | None | Ethernet on some; no PoE | $35–55 | dev platform |
| 7 | **ESP32-P4 (Waveshare P4-ETH, Olimex P4-PC)** | ESP-IDF (Apache), 30-month support windows | **Encoder open (Apache)**; ISP algorithms binary | Espressif RTSP closed; one hobby beta | None integrated | Ethernet, PoE boards | ~$35 | re-evaluate 2028 |
| – | Axis / Hanwha (closed, for reference) | Closed | Closed | Closed | Built in | PoE | $160–540 | 10 y of vendor patches |

## Pros and cons

### 1. OpenIPC on a SigmaStar or HiSilicon/Goke PoE bullet
- **Pros:** real camera hardware (IR-cut, IR LEDs, weatherproof, PoE, hardware
  H.264/H.265); RTSP, ONVIF, motion, automatic night mode; broadest SoC
  coverage; paid support and commercial FPV customers; SigmaStar is the
  reflash-friendly volume leader (37% share, unsanctioned, ships OpenIPC from
  the factory in RunCam/Emax products).
- **Cons:** Majestic is a proprietary binary fetched from S3 without a hash;
  vendor kernel and ISP blobs never patched; userland pinned to Buildroot
  2024.02 with dropbear 2022.82; 2026 firmware commits are one maintainer plus
  AI-agent accounts; roadmap is FPV-only; no vetted camera list (read the SoC
  marking, flash over UART); sysupgrade verifies md5 only.
- **Verdict:** best "buy a real camera" open route. Five-year consumable on a
  no-WAN VLAN. Mirror the release assets locally.

### 2. Thingino on Ingenic T31 cameras
- **Pros:** most open software of any route: GPL streamer, Buildroot 2026.08,
  weekly releases, SD-card install with revert, IR-cut/IR-LED control, MQTT,
  open ISP driver in progress; rebuilds offline forever; bus factor 2.
- **Cons:** hardware is WiFi-heavy, no PoE models; supply closing (Wyze v3 no
  longer sold, T41 secure boot, Tapo C500 bootloader locked Jan 2026);
  Ingenic blobs on a 3.10 kernel; no security policy; hobby funding.
- **Verdict:** best software, dying hardware. Buy stock and spares now; plan
  three to five years.

### 3. Raspberry Pi node (Pi 4 / CM4, or Pi 5)
- **Pros:** the only route with open camera algorithms; production commitments
  to 2034 (Pi 4) and 2036 (Pi 5); Debian security to 2030; hardware H.264 on
  Pi 4; PoE HAT; any sensor (IMX708, IMX462 starlight, OV5647 IR-cut boards);
  full Linux; zero exit cost. **Pi 5 is the only fully blob-free camera
  pipeline in existence**: mainline CSI and ISP drivers, software encode.
- **Cons:** ~€145–170 and 4 W per node; most labour (~12 h/yr); DIY
  weatherproofing; SD wear unless read-only root; Pi 4's ISP and encoder pass
  through closed VideoCore firmware and live in the Pi kernel fork; Pi 5
  costs a CPU core per stream and has no hardware decode; no ONVIF (Frigate
  needs it only for PTZ); third-party IR-cut boards have no lifecycle promise.
- **Verdict:** the best genuinely open platform. Use for the positions that
  matter, not for six.

### 4. Luckfox Pico Ultra (Rockchip RV1106)
- **Pros:** ~$30 with PoE and eMMC; hardware H.264/H.265 at 4 MP; small NPU;
  open Buildroot SDK; OpenIPC support merged Sep 2026; mainline platform
  patches landing.
- **Cons:** vendor 5.10 kernel (LTS EOL Dec 2026) with rockit/rkaiq blobs in
  stock and OpenIPC alike; no IR-cut/IR-LED module; bare board; Rockchip
  discontinued the RV1126 after ~5 years; Luckfox founded 2020; ISP and
  encoder on no mainline roadmap.
- **Verdict:** cheapest wired open-userland node; a five-year tinkerer's bet.

### 5. USB UVC IR camera on a Linux host
- **Pros:** cannot phone home by construction (no CPU, no network stack);
  IR-cut, IR LEDs, IP66 housings, on-camera H.264 variants; standard kernel
  driver forever; ~3 h/yr maintenance.
- **Cons:** ~5 m USB; extenders are the least reliable component; one host
  per few cameras; no PoE; H.264 UVC models vanish silently.
- **Verdict:** right for one or two positions within reach of the NVR.

### 6. Seeed reCamera
- **Pros:** fully open hardware (KiCad), 1 TOPS NPU, open SDK, RISC-V
  mainline work, $35–55.
- **Cons:** no IR, no weatherproofing, no PoE, DIY RTSP, tiny ecosystem.
- **Verdict:** development platform, not a mounted camera.

### 7. ESP32-P4
- **Pros:** hardware H.264 1080p30 with an Apache-licensed encoder driver,
  Ethernet, PoE boards, no WiFi on die, 12-year Espressif supply, ~2 W.
- **Cons:** ISP algorithms binary, RTSP stack closed, only a one-person
  WiFi-only beta firmware, no IR-cut integration, ESP-IDF rebase every 30
  months.
- **Verdict:** re-evaluate 2028.

## The pick

No single winner, because openness and supply pull apart. For cameras you
mount outside and forget: **OpenIPC on SigmaStar PoE bullets**. For the
platform that is open all the way down: **a Raspberry Pi node**, Pi 4 for
hardware encode, Pi 5 for zero blobs. Thingino is the software you wish ran
on the other two; USB fills the gap next to the NVR. If firmware-support
years outrank auditability, Axis and Hanwha beat all of these at three times
the price, with closed firmware.
