# 02 — ESP32 cameras: night vision, firmware, and where they actually fit

Research date: 2026-09-16. Sources are mostly GitHub READMEs and issues; several
vendor sites were unreachable from the research sandbox, and anything that
rests on a search snippet alone is marked *(snippet only)*.

## Verdict

**Does "ESP32 + camera" have night vision? No, not as sold.** The stock
AI-Thinker ESP32-CAM and every mainstream ESP32-S3 camera board is a fixed-lens,
daylight camera. To see at night you need three things it does not ship with:

1. A lens/sensor module **without the IR-cut filter** (sold as "850nm" or
   "night vision" OV2640 drop-in modules).
2. **Your own IR illuminator** (850 nm LEDs: faint red glow, more sensitive;
   940 nm: invisible, weaker).
3. Ideally a **switchable IR-cut filter** so daytime colour is not ruined.
   Only one board found documents this: LilyGO T-CameraPlus-S3 (AP1511B IR-cut
   driver). No board found ships IR LEDs.

The onboard "flash" on GPIO4 of the AI-Thinker board is a **white visible LED**,
not IR. Every firmware treats it as a lamp.

Beyond night vision, the class has a structural limit for CCTV: **no hardware
H.264 on ESP32 or ESP32-S3.** All firmware streams MJPEG, so the NVR must
transcode each stream on its CPU. Realistic output is 10 to 25 fps at VGA/SVGA
and about 5 fps at UXGA or 1080p. Boards are WiFi-only apart from a few SPI
Ethernet (W5500) boards. Power brownouts and sensor heat are the recurring
failure modes.

**Where it fits:** a cheap auxiliary camera for a spot where you would not
otherwise have one (mailbox, shed, indoor room), on its own SSID, with the
rest of the system built on wired open-firmware cameras (see
`03-open-camera-firmware.md`). Not the backbone.

**ESP32-P4 is the first Espressif part that fits the job on paper**: hardware
H.264 1080p30, MIPI-CSI with ISP, 100 Mbit Ethernet MAC, no WiFi radio on the
chip itself. One community project already streams 1080p30 H.264 RTSP over
Ethernet on it. But as of Sep 2026 there is no turnkey open firmware, no ONVIF,
Espressif's own RTSP stack is a closed binary, and no PoE P4 board could be
verified. Developer-grade, not turnkey.

## Firmware projects (status as of 2026-09-16)

| Project | License | Last activity | Status | What it gives you |
|---|---|---|---|---|
| [espressif/esp32-camera](https://github.com/espressif/esp32-camera) | Apache-2.0 | 2026-09-14 | Active | The sensor driver everything else uses. ESP32/S2/S3 only. OV2640, OV3660, OV5640, OV7670, OV7725, GC-series, SC-series, HM1055. PSRAM required above CIF. |
| [s60sc/ESP32-CAM_MJPEG2SD](https://github.com/s60sc/ESP32-CAM_MJPEG2SD) | AGPL-3.0 | 2026-08-29 (v10.9.5a) | Active, 0 open issues | **The most complete open firmware.** MJPEG + RTSP (video, audio, subtitles), on-device motion detection, SD/AVI recording, FTP/WebDAV, MQTT/Home Assistant. Supports AI-Thinker, Freenove S3, XIAO Sense, Waveshare ESP32-S3-ETH (Ethernet, PoE variant). Needs 4 MB PSRAM (ESP32) / 8 MB (S3). |
| [rzeldent/esp32cam-rtsp](https://github.com/rzeldent/esp32cam-rtsp) | MIT | 2026-09-02 | Active | RTSP `rtsp://ip:554/mjpeg/1` + HTTP stream/snapshot, 16+ board presets, OTA. Open issues about RTSP lag at VGA and Frigate not receiving frames. Ethernet requested, not done. |
| [rjsachse/ESP32-RTSPServer](https://github.com/rjsachse/ESP32-RTSPServer) | MIT | 2025-08-27 | Slow but maintained | Arduino RTSP library with I2S audio, UDP/TCP/multicast. ~50 fps QVGA / 5 fps UXGA on S3+OV2640. |
| [yoursunny/esp32cam](https://github.com/yoursunny/esp32cam) | ISC | 2026-07-26 | Active | Clean Arduino wrapper with MJPEG examples. |
| [Tasmota32 webcam build](https://github.com/arendst/Tasmota) | GPL-3.0 | v15.6.0, 2026-08-25 | Active | `tasmota32-webcam`: MJPEG HTTP + RTSP (via Micro-RTSP, port 8554), on-device motion detection by frame differencing, OV2640 controls only. Falls back to VGA without PSRAM. |
| [ESPHome esp32_camera](https://github.com/esphome/esphome) | MIT/GPL | 2026.9.0, 2026-09-16 | Active | Native Home Assistant camera + optional MJPEG web server (one stream at a time). Defaults 10 fps. Presets for AI-Thinker, M5Stack, TTGO, ESP-EYE, XIAO Sense, Waveshare S3-ETH. Docs warn M5Stack camera boards overheat. |
| [espressif/esp-video-components](https://github.com/espressif/esp-video-components) | Apache-2.0 + "Espressif MIT" | 2026-09-16 | Active | V4L2-style camera framework for **ESP32-P4** (MIPI-CSI, hardware encode). HTTP video server and UVC examples. **No RTSP example.** |
| [r4d10n/esp32p4-uvc-video](https://github.com/r4d10n/esp32p4-uvc-video) | Apache-2.0 | 2026-02-16 | Young (22 stars) | **Closest thing to an open P4 IP camera.** Olimex ESP32-P4-DevKit + OV5647: UVC webcam and RTSP H.264 1080p30 over Ethernet via hardware encoder. Single client, UDP only, no audio, tested with VLC/ffplay only. |
| [John-Varghese-EH/ESP32CAM-ONVIF](https://github.com/John-Varghese-EH/ESP32CAM-ONVIF) | not captured | 2026 | Beta (65 stars) | RTSP + **ONVIF Profile S** on ESP32/S3 (MJPEG) and **ESP32-P4-Function-EV with hardware H.264 1080p30**. Bundles optional Google Drive / Telegram extras; leave them disabled. |
| [espressif/esp-webrtc-solution](https://github.com/espressif/esp-webrtc-solution) | Non-OSI "Espressif Modified MIT" | 2026-09-08 | Active | P4 hardware H.264 over WebRTC/WHIP. Browser-facing, not RTSP to an NVR. Licence restricts use to Espressif silicon. |
| esp_media_protocols (esp-adf-libs) | Espressif, binary | – | Closed | Espressif's RTSP/RTMP server. Ships as `.a` + headers. A P4 camera RTSP example was requested in Apr 2025 with no visible answer. |
| [geeksville/Micro-RTSP](https://github.com/geeksville/Micro-RTSP) | MIT | 2023-09 | Dormant | MJPEG-over-RTSP library Tasmota builds on. H.264 request from 2020 never answered. |
| [easytarget/esp32-cam-webserver](https://github.com/easytarget/esp32-cam-webserver) | LGPL-2.1 | EOL announced 2024-12-06 | **Dead** | Historic reference only. |
| [arkhipenko/esp32-cam-mjpeg-multiclient](https://github.com/arkhipenko/esp32-cam-mjpeg-multiclient) | BSD-3 | 2020-08 | **Dead** | Multi-client MJPEG demo. |
| [ArduCAM/ArduCAM_ESP32S_UNO](https://github.com/ArduCAM/ArduCAM_ESP32S_UNO) | – | 2021-06 | **Dead** | Discontinued board. |

If you build with ESP32 today, pick **s60sc/ESP32-CAM_MJPEG2SD** for a
standalone camera or **ESPHome** if the rest of the house is already on Home
Assistant. Everything else is a library or a demo.

## Night vision details

- **Stock lens has a fixed IR-cut filter.** Standard modules are blind under IR
  at night; night vision needs a module without the filter or an "850nm"
  variant *(snippet only: rntlab.com question "ESP32-CAM with no IR filter")*.
  No primary datasheet could be opened to confirm this, but it matches every
  vendor listing seen.
- **Drop-in replacement modules.** 24-pin OV2640 modules for ESP32-CAM are sold
  with "650nm" (normal, IR-cut) or "850nm" (IR-pass) lenses and 66° to 222°
  fields of view *(snippet only: esp32s.com, mikroelectron.com, AliExpress)*.
  They give IR sensitivity but no illumination.
- **IR illumination is on you.** 850 nm is the practical choice (more sensor
  response, slight visible glow). 940 nm is invisible but needs roughly twice
  the LED power for the same image. Power the LEDs from the 5 V rail with their
  own regulator; pulling them from the ESP32-CAM's 3.3 V will brown out the
  board.
- **Switchable IR-cut.** Only LilyGO
  [T-CameraPlus-S3](https://github.com/Xinyuan-LilyGO/T-CameraPlus-S3)
  (OV2640, AP1511B IR filter driver, 8 MB PSRAM, GPL-3.0, hw v1.2 2025-04) has
  one. No IR LEDs on it either.
- **Other S3 boards, none with IR hardware:** Freenove ESP32-S3 WROOM CAM
  (N8R8, 8 MB PSRAM), Seeed XIAO ESP32S3 Sense (OV2640/OV3660), LilyGO
  LilyGo-Cam-ESP32S3 (PIR on GPIO17), M5Stack Timer Camera (OV3660, 3 MP, 66.5°,
  RTC; overheats per ESPHome docs), Espressif ESP32-S3-EYE (OV2640, 8 MB octal
  PSRAM).
- **Sensors:** OV2640 (2 MP) is the default; OV3660 (3 MP) and OV5640 (5 MP)
  work on S3 via DVP but OV5640 modules overheat without a heat sink because the
  board feeds their internal 1.5 V regulator too much voltage (s60sc README).
  On P4, MIPI sensors SC2336 (1080p), SC202CS, OV5647, OV5640/45, OV2710, OV9281
  (mono) are in the esp_cam_sensor list. Whether a given module has an IR-cut
  filter depends on the module vendor.
- **No quantitative low-light data** was reachable. Treat OV2640 night images as
  noisy and low-fps.

## ESP32-P4 details

- Dual-core RISC-V 400 MHz, up to 32 MB PSRAM, hardware H.264 (1080p30 verified
  by r4d10n) and hardware JPEG (1080p encode 26 fps per ESP-IDF docs), 2-lane
  MIPI-CSI with ISP, DVP, internal EMAC (RMII, external PHY, 100 Mbit). No WiFi
  on the P4 die; boards add an ESP32-C6.
- Boards verified from Espressif/Olimex docs: **ESP32-P4-Function-EV-Board**
  (RJ45, MIPI-CSI with 2 MP camera, 7" display, ESP32-C6),
  **ESP32-P4-EYE** (OV2710 2 MP, LCD, mic, microSD, no Ethernet, white fill
  light, YOLOv11-nano demo), **Olimex ESP32-P4-DevKit** (CERN-OHL-S hardware,
  IP101GR PHY). Search-only, pages blocked: Waveshare ESP32-P4-NANO / P4-ETH /
  P4-Pico, Spotpear ESP32-P4-C6, DFRobot P4 kit. **PoE on any P4 board: not
  verified.**
- Software gap: esp-video has HTTP and UVC examples, no RTSP. Espressif's RTSP
  is closed. WebRTC solution is non-OSI licensed. The only open end-to-end P4
  IP camera is r4d10n/esp32p4-uvc-video, single-client, untested against any
  NVR.

## Feeding MJPEG cameras to an NVR

- Espressif's own FAQ: "ESP32-S3 only supports MJPEG encoding, but H264/H265
  format encoding is needed when implementing rtsp/rtmp streaming"
  ([esp-faq](https://github.com/espressif/esp-faq/blob/master/docs/en/application-solution/camera-application.rst)).
  Their software H.264 for S3 does 17 fps at 320x192. Unusable.
  **Correction from the long-term pass:** the P4 *hardware* encoder path in
  [esp-h264-component](https://github.com/espressif/esp-h264-component) is C
  source under Apache-2.0. The ISP algorithms (`esp_ipa`) remain prebuilt
  binaries. See `07-long-term.md`, section 6.
- Frigate docs: MJPEG cameras "require encoding the video into h264 for
  recording and restream roles... This will use significantly more CPU"
  ([camera_specific.md](https://github.com/blakeblackshear/frigate/blob/dev/docs/docs/configuration/camera_specific.md)).
  Recommended pattern:

  ```yaml
  go2rtc:
    streams:
      shed_cam: "ffmpeg:http://192.168.50.21/stream#video=h264#hardware"
  cameras:
    shed_cam:
      ffmpeg:
        inputs:
          - path: rtsp://127.0.0.1:8554/shed_cam
            roles: [detect, record]
  ```

  Budget roughly one CPU core per MJPEG camera transcoded in software, or use
  the NVR's iGPU (`#hardware`) and budget much less.
- History of pain: Frigate discussions from 2022 are full of ffmpeg "Failed to
  transfer data to output frame" and green frames from ESP32 MJPEG; a working
  config was confirmed Oct 2024
  ([discussion 14255](https://github.com/blakeblackshear/frigate/discussions/14255));
  go2rtc issue 292 shows ffmpeg crashing on an ESPHome MJPEG endpoint. Expect
  to tune.

## Practical limits (measured by the firmware authors)

| Board / sensor | Resolution | fps | Source |
|---|---|---|---|
| ESP32 + OV2640 | ≤240x240 | ~45 | s60sc |
| ESP32 + OV2640 | QVGA to HVGA | ~40 | s60sc |
| ESP32 + OV2640 | VGA to UXGA | 5 to 20 | s60sc |
| ESP32-S3 + OV2640 | any | ~2x ESP32 | s60sc |
| ESP32-S3 + OV2640 | QVGA / UXGA | 50 / 5 | rjsachse |
| ESP32-S3 + OV5640 | FHD to QSXGA | 4 to 6 | s60sc |
| ESPHome default | any | 10 | esphome docs |

- **Wired options:** Waveshare ESP32-S3-ETH (W5500 SPI Ethernet, optional
  802.3af PoE module, OV2640/OV5640 header) is supported by s60sc and ESPHome
  *(specs from listings only)*. Everything else is WiFi. S3 has no Ethernet MAC.
- **Heat:** OV5640 needs a heat sink. M5Stack camera boards overheat. Use a
  vented case.
- **Stability:** "Crash loop detected" is almost always the power supply
  (s60sc). USB ports do not supply enough current. Use a real 5 V / 2 A supply
  and enable the firmware watchdog or a scheduled reboot.
- **PSRAM:** mandatory. Anything above CIF JPEG needs it; ESPHome refuses
  without it; s60sc needs 4 MB (ESP32) / 8 MB (S3).
- **Power draw:** commonly quoted ~180 mA at 5 V streaming, ~310 mA with the
  white LED. Not verified this round; spec-sheet hosts were blocked.

## Not verified this round

- Primary-source confirmation of the IR-cut filter on the stock lens, and the
  exact spectral behaviour of "850nm" drop-in modules.
- Any ESP32 board shipping IR LEDs, or an automatic day/night IR-cut add-on for
  the AI-Thinker board.
- XIAO ESP32S3 Sense and Freenove S3 sensor details (vendor docs blocked).
- Waveshare / Spotpear / DFRobot P4 board specs and any PoE option.
- ESP32-CAM power figures and any measured low-light comparison.
