# 05 — DIY camera hardware beyond ESP32: Raspberry Pi, Luckfox, USB cameras

Research date: 2026-09-16. Legend: **verified** = primary page fetched;
*(snippet)* = search-result text only, vendor page blocked; **[price unverified]**
= launch/list price from memory, every retailer page was blocked.

## Verdict

Only one cheap Linux camera board has all four of: hardware H.264 encode,
onboard Ethernet, a working stock RTSP daemon, and an actively maintained open
camera OS. That is the **Luckfox Pico (Rockchip RV1106)** family, and OpenIPC
merged working support for it on **2026-09-07**. The Pico Ultra adds
**802.3af PoE**. Nobody sells an IR-cut + IR-LED module for it yet, so night
vision means wiring a coil and LED board to GPIO yourself.

The **Raspberry Pi** route gives you the most control and the best documented
software, but costs more per camera than a commercial PoE camera, and the
**Pi 5 has no hardware H.264 encoder** (verified four ways below). Use a Pi 4
or Zero 2 W as a camera node, keep the Pi 5 for the NVR.

**USB UVC cameras with IR** attached to a Pi or mini PC are the one option that
is spyware-proof by construction: no CPU running vendor firmware with a network
stack. The cost is cable length and host count.

## Raspberry Pi camera nodes

### Official camera modules (verified from Pi's hardware spec docs)

| Module | Sensor | Resolution | FoV | Video modes | NoIR variant |
|---|---|---|---|---|---|
| Camera Module 3 / **3 NoIR** | Sony IMX708 | 4608x2592 | 66°x41° | 2304x1296p56, 1296p30 HDR, 1536x864p120 | Yes |
| Camera Module 3 Wide / **Wide NoIR** | IMX708 | same | 102°x67° | same | Yes |
| Camera Module 2 / **2 NoIR** | Sony IMX219 | 3280x2464 | 62°x49° | 1080p47, 640x480p206 | Yes |

- **NoIR = no IR-cut filter, permanently.** IR-sensitive around the clock, so
  daytime colour is washed out and pinkish. No official module has a switchable
  filter and none ships IR LEDs. You add an 850 nm illuminator.
  Prices **[unverified]**: CM3 / CM3 NoIR $25, Wide $35.
- Pi 5, all Pi Zero models and Compute Module IO boards use the 22-pin mini
  connector; buy the Standard-Mini cable. Pi 5 has two camera connectors.
- **Switchable IR-cut with IR LEDs: Arducam** 5 MP OV5647 module with motorised
  IR-cut filter and two IR-LED boards, switching automatically by photoresistor,
  M12 lens mount, listed for Pi 5/4/3 *(snippet, arducam.com blocked)*. OV5647
  is a stock libcamera sensor so the normal rpicam/MediaMTX stack works. The
  170° version with case is marked discontinued. **[price unverified]** $30–45.
  MediaMTX warns that Arducam models needing a custom libcamera (Pivariety,
  64 MP) do not work with its precompiled binaries; plain OV5647/IMX708 is fine.

### Pi 5 has no hardware H.264 encoder (verified)

1. Pi docs `rpicam_vid.adoc`: "Raspberry Pi 5 uses software video encoders."
2. Pi docs `bcm2712.adoc` lists only HEVC hardware decode and costs H.264 as
   CPU work: "H264 1080p30 encode (from ISP) ~30–40% CPU". `bcm2711.adoc`
   (Pi 4) lists "H.264 (1080p60 decode, 1080p30 encode)".
3. rpicam-apps `encoder/encoder.cpp` only creates the hardware encoder on
   `Platform::VC4`; otherwise "No hardware codec available, use x264 through
   libav."
4. picamera2 swaps `H264Encoder` for `LibavH264Encoder` when no hardware
   encoder is present.

### Which Pi as a camera node

| Host | HW H.264 | Ethernet / PoE | Notes | [price unverified] |
|---|---|---|---|---|
| **Pi Zero 2 W** | Yes, VideoCore IV 1080p30 | None; 2.4 GHz WiFi. PoE via Waveshare PoE/ETH/USB hub HAT *(snippet)* or PoE splitter + USB Ethernet | 512 MB RAM. MediaMTX rpiCamera at 720p / 1.5 Mbps reported to work well ([Tyicenet guide](https://github.com/Tyicenet/Raspberry-Pi-RTSP-Camera-Setup-with-MediaMTX)). Cheapest Pi node. | $15 |
| **Pi 4** | Yes, 1080p30 | GbE, official PoE+ HAT | Best real Pi camera node: wired, PoE, hardware encode. | $45 (2 GB) |
| Pi 5 | **No** | GbE, third-party PoE HATs; official PoE+ HAT+ status **[unverified]** | Poor camera node (cost, heat, software encode). Good NVR host. | $60–65 (4 GB) |

### Streaming stack (verified)

**MediaMTX** has a native `rpiCamera` source. Minimal config:

```yaml
paths:
  cam:
    source: rpiCamera
    rpiCameraWidth: 1920
    rpiCameraHeight: 1080
    rpiCameraFPS: 25
    rpiCameraBitrate: 4000000
    rpiCameraCodec: auto      # hardwareH264 on Pi 4 / Zero 2 W, softwareH264 on Pi 5
    rpiCameraSecondary: true  # low-res M-JPEG substream for detection
```

RTSP at `rtsp://<pi>:8554/cam`, WebRTC on port 8889. Supports Bookworm and
Trixie, 32 and 64 bit. MIT license, ~20k stars
([docs](https://github.com/bluenviron/mediamtx/blob/main/docs/3-publish/14-raspberry-pi-cameras.md)).

go2rtc has no native libcamera source; it uses `exec:rpicam-vid -t 0 --inline -o -`
(add `--libav-format h264` on Pi 5). Its hardware-acceleration wiki covers
Pi 3/4 and does not mention Pi 5.

### Cost per Pi camera (all prices unverified estimates)

| Build | Approx. total |
|---|---|
| Zero 2 W + CM3 NoIR + PoE hub HAT + card + enclosure | $90–120 |
| Zero 2 W WiFi build, no PoE | $60–75 |
| Pi 4 2 GB + PoE+ HAT + Arducam IR-cut/IR-LED + card + enclosure | $140–170 |
| Pi 5 4 GB + PoE HAT + cooler + camera + enclosure (software encode) | $160–190 |

Every option costs more than a mid-range commercial PoE camera. The Pi path
buys full control of the software, not savings.

### Weatherproofing *(snippets)*

Entaniya All-Weather Case WC-01 (O-ring dome for Pi V2/V3 modules, "IP67
equivalent"), Sixfab IP65 enclosure, or a DIY IP65 junction box with a window
and cable gland. Practical: keep IR LEDs outside the dome or window (internal
reflections wash out the image), use flat glass with IR, add a desiccant or vent
plug, and remember a Pi 4/5 dissipates several watts in a sealed box in sun.

## Cheap Linux camera-SoC boards

| Board | SoC / RAM | HW encode | Camera | Ethernet / PoE | Open firmware | [price] | Night vision |
|---|---|---|---|---|---|---|---|
| **Luckfox Pico Pro / Max** | Rockchip RV1106, 128/256 MB | H.264/H.265, 4 MP@30 | 2-lane MIPI, SC3336 3 MP ($9), MIS5001 5 MP | 100 M, no PoE | Buildroot SDK (kernel 5.10.160) **verified**; stock `rkipc` RTSP at `rtsp://<ip>/live/0` *(snippet)*; **OpenIPC rv1106 support: initial merge 2025-03-21 (PR 1761), PR 2382 merged 2026-09-07 tested on Pico Max + SC3336, "majestic streams h264 2304x1296@20fps"** **verified** | $14 / $16 (2024) | None sold. Drive IR-cut coil + LEDs from GPIO. |
| **Luckfox Pico Ultra / Ultra W** | RV1106G3, 256 MB, 8 GB eMMC | same | 2-lane MIPI | 100 M + **802.3af PoE module**; WiFi 6 on W | Same SDK; OpenIPC eMMC path untested in the PRs read | [unverified], sold as "PoE Kit" | same |
| Luckfox Pico Mini B / Pico / Plus | RV1103, 64 MB | yes | MIPI | Plus has RJ45 | Same SDK; ~34 MB RAM free | ~$9 | none |
| Milk-V Duo S | Sophgo SG2000, 512 MB | H.264/H.265 | 2x MIPI, GC2083/GC4653 | 100 M, WiFi 6, no PoE | duo-buildroot-sdk **verified**; `middleware/v2/sample` has venc/vio but **no RTSP sample**; Sophgo `cvi_rtsp` sparsely maintained; **no OpenIPC port** | ~$11–15 | none |
| Milk-V Duo | CV1800B, 64 MB | yes | MIPI | Ethernet via add-on | same | ~$5–9 | none |
| Sipeed LicheeRV Nano / MaixCAM | SG2002, 256 MB | 2K@30 | MIPI GC4653 | Ethernet via add-on on E/WE | MaixPy `rtsp.Rtsp()` serves `rtsp://<ip>:8554/live` **verified**; plain-Linux RTSP question (issue 97, Aug 2025) unanswered | [unverified] | none |
| Radxa Zero 3W / 3E | RK3566, 1–8 GB | 1080p60 | 22-pin MIPI | 3E: GbE | Debian + rkmpp; no ready camera firmware; GStreamer `mpph264enc` + MediaMTX DIY | ~$15–30 | none |
| Orange Pi Zero 3 | Allwinner H618 | Encoder in silicon but **unusable under Linux** *(snippet)* | **No CSI**, USB only | GbE | software encode only | ~$20–35 | via USB IR camera |
| Banana Pi BPI-M4 Zero | H618 | same problem, CPU pegs at 100% *(forum snippet)* | CSI (Android demo) | GbE | none | [unverified] | none |

## USB UVC cameras with IR on a Pi or mini PC

- **ELP** sells UVC modules with automatic IR-cut (photoresistor) and 850 nm
  IR LEDs: 2 MP OV2710 1080p (MJPEG/YUY2), **on-camera H.264** variants
  (AR0330), IP66 dome and bullet housings *(snippets)*. **[price unverified]**
  $35–70.
- **Pros:** no network stack, no vendor firmware to audit, physically cannot
  phone home. Host owns encoding, recording and exposure. Works with go2rtc,
  MediaMTX and Frigate via V4L2/ffmpeg.
- **Cons:** USB 2.0 cable limit about 5 m; beyond that, active repeaters or
  USB-over-Cat5e extenders (50 m, externally powered, some limited to two
  cameras) *(snippets)*. All cameras on one root hub share 480 Mbps and UVC
  reserves isochronous bandwidth, so realistically 2 to 4 MJPEG 1080p cameras
  per host. Uncompressed 1080p30 (about 750 Mbps) is impossible. Buy the H.264
  UVC variants to skip host encoding.

## ESP32-P4 (see `02-esp32-cameras.md` for the full picture)

Hardware fits: MIPI-CSI + ISP, hardware H.264 1080p30, internal Ethernet MAC,
**no WiFi on the die** (wired-only by default, which is what you want).
Firmware is hobby-grade: [r4d10n/esp32p4-uvc-video](https://github.com/r4d10n/esp32p4-uvc-video)
(RTSP H.264 1080p30, 22 stars) and
[John-Varghese-EH/ESP32CAM-ONVIF](https://github.com/John-Varghese-EH/ESP32CAM-ONVIF)
(RTSP + ONVIF Profile S, P4 hardware H.264, "Beta", 65 stars, bundles optional
Google Drive / Telegram extras you would leave disabled). No IR-cut or IR-LED
integration anywhere.

## Not verified this round

- All prices.
- Arducam IR-cut module details (arducam.com blocked).
- Official Pi 5 PoE+ HAT+ shipping status.
- Luckfox Pico Ultra eMMC boot under OpenIPC.
- UniFi, USB cable and extender specifics are general knowledge.
