# 08 — ESP32 starter kit: what to buy, what to skip, first steps

Written 2026-09-18. Purpose: a cheap first camera to learn the whole
pipeline (camera → isolated VLAN → NVR → remote view) while the durable core
from `07-long-term.md` gets built. It is a learning platform, not a security
camera: MJPEG-only, 2 MP, blind at night as sold. Prices are retailer
snapshots from search results, USD, unverified, September 2026.

## Buy this: the wired kit (recommended)

| # | Item | Why | ~Price | Verified? |
|---|---|---|---|---|
| 1 | **Waveshare ESP32-S3-POE-ETH-CAM-KIT** (ESP32-S3-ETH board + PoE module + OV2640 camera) | The only cheap ESP32 camera board with **wired Ethernet and PoE**, so it sits on the camera VLAN with WiFi off, exactly as the threat model wants. Named profile `CAMERA_MODEL_Waveshare_ESP32_S3_ETH` in the best firmware, which states "PoE variants are supported at the hardware level". ESP32-S3 with 8 MB PSRAM (required). | $22 AliExpress, $25 Waveshare store, $26–35 Amazon ([search](https://www.amazon.com/ESP32-S3-Development-Ethernet-Processor-ESP32-S3-POE-ETH-CAM-KIT/dp/B0DKNRH7HF)) | Firmware support: **verified** (s60sc README). Kit variants and prices: search snippets. PoE standard (802.3af per first pass) and W5500 SPI Ethernet: snippets, Waveshare wiki blocked. |
| 2 | **OV2640 "850nm" night-vision lens module, 24-pin DVP, 0.5 mm pitch**, 120° or 160° | The stock lens has a fixed IR-cut filter and is blind under IR. This drop-in module has no IR-cut, so it sees the illuminator. Daytime colour goes pinkish; accept that on a learning camera. | $5–10 module alone ([mikroelectron](https://mikroelectron.com/product/me-13581), [esp32s.com](https://esp32s.com/product/24pin-ov2640-camera-module-for-esp32-cam-camera-module-2mp-180-66-120-160-222-200-degree-650nm-850nm-night-vision-dvp/), [Amazon](https://www.amazon.com/OV2640-Camera-Module-Degree-Vision/dp/B0DNYKDZND)) | Modules exist: verified from listings. **Connector match with the Waveshare board: not verified from a primary page.** Both are described as 24-pin 0.5 mm DVP; check the Waveshare wiki photo before ordering. |
| 3 | **850 nm IR illuminator, 12 V, 8 to 12 LEDs, IP67, built-in photocell**, with its own 12 V 2 A adapter | The board has no IR LEDs. A CCTV-style illuminator with a photocell switches itself at dusk. **Never power it from the ESP32's 5 V rail**; it must have its own supply. | $15–30 ([Walmart example](https://www.walmart.com/ip/IR-Illuminator-8-LED-Long-Range-Outdoor-Use-Infrared-Light-Night-Vision-850nm-12V-Waterproof-floodlight-CCTV-Cameras-IP-Security-Camera/1262566598), [Amazon 48-LED](https://www.amazon.com/clp/B0DNFQQ1D5)) | Listings only. |
| 4 | **IP65 junction box** with a flat glass or acrylic window, plus one cable gland | Weatherproofing. Flat window, not a dome, because IR reflects inside domes. Mount the illuminator **outside** the box. Add a silica-gel pack. | $8–15 | General practice. |
| 5 | **microSD card, 16–32 GB, A1 class** (optional) | s60sc can record locally and buffer motion clips. Not needed if the NVR records, which is the plan. Skip it to avoid SD wear. | $6–8 | – |
| 6 | **PoE switch or a single-port 802.3af injector** | You need a PoE source. The switch is part of the durable core anyway; an injector is the cheap start. | $15–25 injector | – |

**Wired kit total, no switch: about $60 to $90.**

## Or this: the cheapest possible learning kit (WiFi)

| # | Item | ~Price | Notes |
|---|---|---|---|
| 1 | **AI-Thinker ESP32-CAM bundle with the 850nm OV2640 module and ESP32-CAM-MB USB programmer board** | $15 ([AliExpress](https://www.aliexpress.com/item/4001054283208.html)) | 4 MB PSRAM; the firmware author notes the ESP32 "cannot support all of the features as it will run out of heap space". WiFi only. |
| 2 | **5 V 2 A power supply and a short, thick USB cable** | $8 | Brownouts are the number one failure. "Crash loop detected" means the supply. USB ports on laptops are not enough. |
| 3 | Same illuminator and box as above | $25–45 | |

**WiFi kit total: about $50 to $70.** It must live on its own SSID mapped to
the camera VLAN with client isolation, per `06-network-isolation.md`.

## Do not buy for this purpose

- **LilyGO T-CameraPlus-S3.** It is the only ESP32 board with a switchable
  IR-cut filter (AP1511B driver), which is tempting, but it is WiFi-only,
  carries a display, speaker and microphone you do not need, its README lists
  only a `Camera_WebServer` example (no RTSP), and it is not named in the
  s60sc board list. Wrong tool for a camera node.
- **OV5640 or OV3660 kits.** The firmware author: "the voltage supply is too
  high for their internal 1.5V regulator, so the camera overheats unless a heat
  sink is applied". No frame-rate gain at CCTV resolutions; no 850 nm variant.
- **M5Stack camera boards.** ESPHome docs warn they overheat over time.
- **No-name boards marked "ESPS3 RE:1.0".** The s60sc README says to avoid
  them.
- **Seeed XIAO ESP32S3 Sense.** Fine board, but WiFi-only and no advantage over
  the Waveshare kit for this job.
- **ESP32-P4 boards, for now.** The Waveshare ESP32-P4-ETH with PoE is about
  $24 and has hardware H.264, but the only active firmware is a WiFi-only
  one-person beta still finishing RTP packetisation. Buy one later if you want
  to tinker at the frontier; it is not a starter.

## Firmware

Use **[s60sc/ESP32-CAM_MJPEG2SD](https://github.com/s60sc/ESP32-CAM_MJPEG2SD)**
(AGPL-3.0, active, v10.9.5a Aug 2026). Facts from its README:

- Select `CAMERA_MODEL_Waveshare_ESP32_S3_ETH` (or `CAMERA_MODEL_AI_THINKER`).
- Ethernet modes: "Standard Ethernet (WiFi off)" or "Eth+AP". Use the first.
- RTSP needs the separate
  [rjsachse/ESP32-RTSPServer](https://github.com/rjsachse/ESP32-RTSPServer)
  library, "version 1.3.1 or above". URI `rtsp://<ip>:<port>`, optional
  user:pass. Single client by default; multicast or the
  `OVERRIDE_RTSP_SINGLE_CLIENT_MODE` define allows more "but may reduce
  performance".
- OV2640: "Maximum 50fps at most resolutions; 25fps at VGA and above".
- On-board white lamp on pin 4 (ESP32) / 48 (Freenove); XIAO has none. It is
  not IR. You can ignore it.

Alternative if the house already runs Home Assistant: **ESPHome** with the
`esp32_camera` component, which has a preset for the Waveshare ESP32-S3 ETH.
Default 10 fps, one MJPEG stream at a time.

## First steps, in order

1. **Flash on the bench with the stock lens.** USB-C, Arduino IDE or
   PlatformIO per the s60sc README. Confirm the web UI and a JPEG snapshot.
2. **Swap the lens module to the 850nm one.** Power off first. The FPC
   connector latch is fragile.
3. **Set Ethernet mode, WiFi off, static IP** in the camera VLAN range, gateway
   blank or pointing at a non-routing address, NTP pointed at your router.
4. **Set 800x600 at 10 fps, JPEG quality around 12.** Higher costs the NVR
   CPU for no detection benefit.
5. **Enable RTSP.** Test with VLC from the NVR host: `rtsp://<ip>:554`.
6. **Prove isolation** before mounting: from the router, run the tcpdump line
   in `06-network-isolation.md` for an hour; the only packets leaving the
   camera VLAN must be to the NVR and NTP.
7. **Add it to Frigate** through go2rtc so the MJPEG becomes H.264 once, on
   the NVR's iGPU:

   ```yaml
   go2rtc:
     streams:
       shed: "ffmpeg:rtsp://192.168.50.21:554#video=h264#hardware"
   cameras:
     shed:
       ffmpeg:
         inputs:
           - path: rtsp://127.0.0.1:8554/shed
             roles: [detect, record]
       detect:
         width: 800
         height: 600
         fps: 5
   ```

   Without `#hardware` the transcode is software x264: about 40% of a Pi 4
   per camera, cheap on an N100.
8. **Mount it.** Illuminator outside the box, aimed the same way, on its own
   12 V adapter. Camera window flat, clean, no IR bounce.
9. **Set a scheduled reboot** in the firmware (weekly) and watch the log for
   "Crash loop detected" in the first week.

## What this teaches you before the real cameras arrive

The VLAN and firewall rules, the Frigate and go2rtc config, the transcode
cost of MJPEG, and how a camera behaves with no internet. All of that carries
over unchanged to the Pi 4 node and the OpenIPC bullets in `07-long-term.md`.
The ESP32 itself becomes the mailbox or shed camera afterwards.

## Not verified

- Waveshare ESP32-S3-ETH PoE module standard and output power, and the exact
  camera connector, because waveshare.com and cnx-software.com were blocked.
- Whether the 850nm 24-pin modules seat correctly on the Waveshare board
  (same nominal connector as the ESP32-CAM; check a photo).
- All prices.
- Illuminator quality; the listings are generic. Any 850 nm 12 V unit with a
  photocell and an IP rating does the job.
