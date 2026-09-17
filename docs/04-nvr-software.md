# 04 — NVR, detection and streaming software with zero cloud dependency

Research date: 2026-09-16. Release dates come from GitHub/GitLab Atom feeds
and repo source files (license files, config defaults, Go/Python source).
Legend: **[V]** verified from primary source; *[S]* search snippet only;
*[K]* general knowledge, not re-verified this round.

## Verdict

- **Frigate** is the default for a fully local AI NVR in 2026: MIT, 0.18.0
  shipped 2026-09-12, daily commits, the broadest detector matrix, and a core
  that is genuinely offline. It makes a handful of outbound calls by default
  that you switch off (GitHub version check, go2rtc STUN defaults, one-time
  model downloads for optional features). Frigate+ is optional paid model
  *training*; inference is local and only images you submit leave the box.
- **ZoneMinder** is the mature classic VMS (GPL-2.0, 1.38.4 Aug 2026). Best
  for analog/USB cameras or scriptable non-AI recording. Ships a telemetry
  daemon that geolocates your IP and a daily update check; both must be turned
  off.
- **Viseron** (MIT, 3.6.2 Sep 2026) is the credible second AI-NVR choice.
- **go2rtc** and **MediaMTX** (both MIT) are the streaming primitives. go2rtc is
  embedded in Frigate and Home Assistant. MediaMTX is the cleaner "nothing
  phones home by default" component.
- **Moonfire NVR** (GPL-3.0) is the best ultra-lean continuous recorder, no
  detection. **SentryShot** (Rust, TFLite/Coral) is minimal and slow-moving.
  **Motion/motionEye** are alive again; **motionEyeOS is dead.**
- **Not open or not cloud-free as marketed:** Shinobi (custom non-OSI EULA;
  the GPL "CE" edition died in 2021), Scrypted NVR plugin (closed, paid),
  Kerberos Vault/Hub (closed), Agent DVR/iSpy (closed), Camect (appliance).
- **Detection hardware in 2026:** Google Coral is de-recommended by Frigate and
  its upstream repos are archived. Intel iGPU/NPU via OpenVINO on an N100/N150
  mini PC needs no add-on card and is the default path. Hailo-8/8L is the
  low-power accelerator. CPU-only detection is officially "not recommended".

## Project table

| Project | License | Latest release | Status | Inputs | Detection | Phones home by default? | Pick it if |
|---|---|---|---|---|---|---|---|
| [**Frigate**](https://github.com/blakeblackshear/frigate) | MIT | **0.18.0, 2026-09-12** | Active, daily | RTSP primary; RTMP, HTTP-MJPEG, USB, ONVIF, WebRTC via bundled go2rtc | Objects on edgetpu, hailo, openvino, onnx, tensorrt (Jetson), rknn, memryx, apple_silicon, cpu; face, LPR, semantic search, GenAI | **Yes, small:** GitHub version check at startup; go2rtc STUN when WebRTC is used; one-time model downloads for optional features. All disableable. | You want AI detection and HA integration on 2 to 30 cameras. |
| [**ZoneMinder**](https://github.com/ZoneMinder/zoneminder) | GPL-2.0 | 1.38.4, 2026-08-11 | Active, daily | ffmpeg (RTSP/RTMP/MJPEG), V4L2, analog cards, ONVIF discovery and events | Zone motion built in; objects via external zmesNg (legacy zmeventnotification archived 2026-05-19) | **Yes:** telemetry daemon incl. IP geolocation via ipinfo.io / ip2location.io, plus daily check to update.zoneminder.com. Both switchable in Options → System. | Mixed analog/USB/IP estate, no AI needed. |
| [**Viseron**](https://github.com/roflcoopter/viseron) | MIT | 3.6.2, 2026-09-07 | Active | RTSP via ffmpeg/gstreamer, go2rtc, ONVIF | darknet, edgetpu, hailo, deepstack, codeprojectai, yolo; motion; face; LPR | None documented; may fetch model weights on first run *[K]* | Simpler MIT alternative to Frigate with Coral/Hailo. |
| [**go2rtc**](https://github.com/AlexxIT/go2rtc) | MIT | v1.9.14, 2026-01-19; last code change 2026-03-17 | **Stalled in 2026** (193 open PRs, one maintainer; see `07-long-term.md`) | RTSP, RTMP, MJPEG, ONVIF, WebRTC, HomeKit, V4L2, ffmpeg/exec, 20+ vendor protocols | none | Default ICE servers `stun.cloudflare.com`, `stun.l.google.com`. Set `webrtc: ice_servers: []`. No telemetry. | Everyone: the restream/WebRTC gateway. |
| [**MediaMTX**](https://github.com/bluenviron/mediamtx) | MIT | v1.21.0, 2026-09-09 | Active | RTSP/S, RTMP/S, HLS, SRT, WebRTC, MoQ, native `rpiCamera`. **No MJPEG, no ONVIF.** | none | None. `webrtcICEServers2: []` by default. | Pi camera → RTSP, headless recorder (fMP4 segments, 1 h, delete after 1 d). |
| [**Moonfire NVR**](https://github.com/scottlamb/moonfire-nvr) | GPL-3.0+ | v0.7.32, 2026-08-15 | Active, solo | RTSP H.264 only | **None** | None. No TLS (put behind a proxy). | Pure continuous recording: six 1080p streams on a Pi 2 under 10% CPU. |
| [**SentryShot**](https://codeberg.org/SentryShot/sentryshot) | GPL-2.0+ | v0.3.11, 2026-04-27 | Slow | RTSP | motion, TFLite objects (Coral via libedgetpu) | None known | Minimal Rust NVR on a Pi with a Coral you already own. |
| [**Motion**](https://github.com/Motion-Project/motion) | GPL-2.0 (4.x), GPL-3.0+ (5.0) | 4.7.1, late 2025; 5.0 dev commit 2026-09-11 | Active, slow releases | V4L2, RTSP/RTMP/MJPEG URLs, Pi cam via `libcamerify` (native libcamera in 5.0) | Pixel motion | None | Minimal daemon with hooks, 1 to a few cams. |
| [**motionEye**](https://github.com/motioneye-project/motioneye) | GPL-3.0 | 0.44.0, 2026-06-21 | Active, revived | Motion netcams, remote motionEye, MJPEG | Motion | None (update check is local-only) | Pi hobbyist, web UI, zero AI. |
| motionEyeOS | GPL-3.0 | 2020-10-26 | **Dead** | | | | Use motionEye on Pi OS instead. |
| [Kerberos Agent](https://github.com/kerberos-io/agent) | MIT | v3.12.2, 2026-09-14 | Active | RTSP, ONVIF; one camera per container | Motion only | **Yes:** default `AGENT_MQTT_URI=tcp://mqtt.kerberos.io:1883`. Set `AGENT_OFFLINE=true`. | Per-camera containers on Kubernetes. Vault/Hub are closed. |
| [Bluecherry](https://github.com/bluecherrydvr/bluecherry-apps) | GPL-2.0 server | 3.1.14, 2025-08-31 | Slow; company closed, donation-funded | RTSP, ONVIF events, analog PCIe cards | Motion only | Licensing removed in 3.1.14 | Analog capture cards. |
| [Shinobi](https://gitlab.com/Shinobi-Systems/Shinobi) | **Custom "Shinobi Open Source Software License Agreement", not OSI** | rolling, commit 2026-09-10 | Active, one dev | RTSP, ONVIF, MJPEG, HLS, USB | Motion; objects via separate plugins | None found in sample config; not audited | Only if you accept a licence that requires a paid subscription for any commercial use. **ShinobiCE (GPL) is dead since 2021-02.** |
| [Scrypted](https://github.com/koush/scrypted) | Mixed: server ISC, core Apache-2.0; **NVR plugin closed and paid** | v0.147.0, 2026-09-13 | Active | RTSP, ONVIF, RTMP, HomeKit, vendor plugins | Plugins; "smart detections" in paid NVR | Plugins from npm; optional cloud login; NVR licence validation *[S]* | HomeKit/Alexa bridging. Not a strict-FOSS offline build. |
| [OpenNVR](https://github.com/open-nvr/open-nvr) | AGPL-3.0 core | v0.1.5, 2026-09-10 (first release 2026-07) | New, 97 stars | ONVIF, RTSP (embeds MediaMTX) | Pluggable, YOLOv8 default | Claims default-deny egress (`DEPLOYMENT_MODE=offline`) | Watch. Too young. |
| Agent DVR / iSpy | **Closed source** | | | | | Account for remote/paid | Excluded. |
| Camect | Proprietary appliance | | | | | Cloud relay for remote | Excluded. |

## Frigate: what leaves the box and how to stop it (all verified from source)

| Behaviour | Where | Disable |
|---|---|---|
| GET `api.github.com/repos/blakeblackshear/frigate/releases/latest` at startup | `frigate/config/telemetry.py`, `frigate/stats/util.py`; `telemetry.version_check` default `True` | `telemetry: { version_check: false }` |
| STUN to Cloudflare/Google during WebRTC negotiation | Bundled go2rtc compiled defaults; Frigate's `create_config.py` does not override them. MSE/HLS live view needs no internet. | `go2rtc: { webrtc: { ice_servers: [] } }` or just use MSE |
| One-time model downloads | Semantic search (Jina CLIP from HuggingFace), face recognition and LPR (GitHub), Hailo default model (Hailo Model Zoo), RKNN default (GitHub). "Once cached, models work fully offline." Default OpenVINO ssdlite is bundled. | Leave off, or enable once with internet, then firewall |
| Frigate+ | Nothing sent unless `PLUS_API_KEY` is set. "The only images sent to Frigate+ are the ones you specifically submit." | Don't set the key |
| Web push notifications | Need Google FCM / Mozilla autopush | Use MQTT / Home Assistant notifications instead |
| GenAI | Provider-optional: Ollama / llama.cpp local, or Gemini/OpenAI cloud | Local provider or off |

UI is self-contained: local assets, bundled fonts, no map view.

## Detection hardware, 2026

| Option | Frigate detector | Inference (Frigate hardware docs) | Reality |
|---|---|---|---|
| **Intel iGPU / NPU (OpenVINO)** | `openvino` | N100/N150 iGPU: MobileNetV2 ~15 ms, YOLOv9-s-320 24–30 ms. Core Ultra NPU: ~6 ms / ~11 ms. Arc A310 ~5 ms. | **Nothing to buy** on any 6th-gen+ Intel. Also does hardware decode (VAAPI/QSV). Community N100: six 4K cams with 720p detect at 25–30% total CPU. |
| **Hailo-8 / 8L** (Pi 5 AI HAT+, M.2) | `hailo` | Hailo-8: ~6–7 ms. Hailo-8L: ~10–11 ms. | AI HAT+ 26 TOPS ~$110, 13 TOPS ~$70 *[S]*. Pain point: host driver vs HailoRT version mismatch, fix by rebuilding the driver. **Long-term pass: out-of-tree driver, Hailo-8 on a maintenance branch, company being acquired by Microchip (agreement 2026-07-24). Cache the runtime, buy a spare.** |
| **Google Coral** | `edgetpu` | ssd-mobilenet ~10 ms; quantized YOLOv9 since 0.17 | Frigate: "no longer recommended for new installations". `google-coral/edgetpu` archived 2026-04-19, `libedgetpu` archived 2025-10-14. PCIe driver fails on kernels ≥6.4. **Fine if you own one, don't buy one.** |
| NVIDIA dGPU / Jetson | `onnx` (TensorRT EP) / `tensorrt` (Jetson) | RTX 3070 ~6–8 ms | Overkill for home unless you want GenAI on GPU. |
| Rockchip RK3588/3576/3568/3566 NPU | `rknn` | RK3588: YOLO-NAS-s ~25–30 ms | Orange Pi 5 / Rock 5 ~$120–180 *[S]*. Vendor-kernel dependence. |
| CPU only | `cpu` / `openvino` CPU mode | ~2–3 fps on i7-class | "Not recommended." |

## NVR hosts people actually run

| Host | Cameras | Notes |
|---|---|---|
| **Intel N100/N150 mini PC** (Beelink EQ13 is the Frigate docs' pick; dual NIC for an isolated camera LAN) | 4–8 comfortably; six 4K reported at 25–30% CPU | ~$150–200 *[S]*. EQ13 out of stock late 2025; EQ14 has known compatibility issues (Frigate issue 21009). |
| **Raspberry Pi 5 8 GB + AI HAT+** | ~4–5 streams | Detection is not the bottleneck; **decode is**: Pi 5 has no H.264 hardware decoder (HEVC only). Frigate collaborator: "a mini PC with an iGPU would definitely be an improvement." |
| **Used ThinkCentre Tiny / OptiPlex Micro**, 6th-gen+ Intel | 8–12 | $100–250. M720q (i5-9400T) ran 12 cams at 60–90% CPU until decode was offloaded to the iGPU. |

Frigate sizes by simultaneous activity: 1–6 cams low, 6–12 moderate, 12+ high.
8 GB RAM minimum with semantic search. AVX2 CPU required.

## Recommended stacks

### (a) Reference: home, 4–8 cameras, zero cloud

- **Host:** Intel N100/N150 mini PC or used i5-8500T/9400T Tiny, 16 GB RAM,
  NVMe for OS, 2–4 TB surveillance HDD for recordings. Dual NIC or VLAN so
  cameras have no WAN route (see `06-network-isolation.md`). Debian, bare or
  Proxmox with iGPU passthrough.
- **NVR:** Frigate 0.18 in Docker.

  ```yaml
  telemetry:
    version_check: false
  go2rtc:
    webrtc:
      ice_servers: []
  detectors:
    ov:
      type: openvino
      device: GPU
  ffmpeg:
    hwaccel_args: preset-vaapi
  record:
    enabled: true
    retain:
      days: 5
      mode: motion
    alerts:
      retain:
        days: 30
  ```

  Detect on the camera's H.264 sub-stream (640–1280 px, 5 fps), record the main
  stream. Leave semantic search, face, LPR and GenAI off unless you cache the
  models deliberately. Do not set `PLUS_API_KEY`.
- **Events / dashboard:** local Mosquitto MQTT; Home Assistant with the Frigate
  HACS integration if you want it. HA's managed go2rtc already ships
  `ice_servers: []`.
- **Storage rule of thumb** *[K]*: 8 cams × 4 Mbps continuous ≈ 345 GB/day.
  Motion/alert-tiered retention cuts this 3–5×.
- **Accelerator:** none needed on Intel. Hailo-8L M.2 only if you want the iGPU
  free for enrichments.
- **Alternatives:** Viseron if Frigate feels heavy. ZoneMinder 1.38 + zmesNg
  for analog/USB (decline telemetry, disable CHECK_FOR_UPDATES). Moonfire for
  pure continuous recording.

### (b) Minimal tinkerer: 1–2 ESP32 or Pi cameras

- **No-AI, cheapest:** Pi 4 (or Zero 2 W for one camera) running motionEye
  0.44. Add the ESP32-CAM as a network MJPEG camera and the Pi camera via
  `libcamerify motion`. Motion-triggered recording only.
- **Pi camera as IP camera:** MediaMTX with the native `rpiCamera` source gives
  a hardware-encoded H.264 RTSP stream (Pi 4 / Zero 2 W) that any NVR records
  without transcoding.
- **AI on a Pi:** Pi 5 8 GB + AI HAT+ 13 TOPS + Frigate `hailo` detector. Feed
  it H.264 sources. Expect ~4 streams because the Pi 5 decodes H.264 in
  software.
- **Very small:** SentryShot or Moonfire on a Pi 4.

### (c) MJPEG-only cameras (ESP32-CAM) in Frigate / go2rtc

- Frigate docs: MJPEG cameras "require encoding the video into h264 for
  recording, and restream roles. This will use significantly more CPU."
  Pattern:

  ```yaml
  go2rtc:
    streams:
      mjpeg_cam: "ffmpeg:http://ESP32_IP:81/stream#video=h264#hardware"
  cameras:
    mjpeg_cam:
      ffmpeg:
        inputs:
          - path: rtsp://127.0.0.1:8554/mjpeg_cam
            roles: [detect, record]
  ```

  `#hardware` picks vaapi/qsv/v4l2m2m/cuda/rkmpp; without it the transcode is
  software x264.
- `detect` can decode MJPEG straight to raw frames, but `record` **must** be
  H.264/H.265 ("using h264 or h265 is a requirement", maintainer, Frigate
  issue 13031). Consuming the MJPEG URL directly needs
  `input_args: preset-http-mjpeg-generic`.
- **CPU cost:** software x264 of one ESP32-CAM stream ≈ **40% of a Pi 4** per
  camera (issue 13031). Two streams decode-only ≈ 22% of a Pi 4, and
  `h264_v4l2m2m` gives no benefit because the input isn't H.264 (issue 5839).
  VAAPI MJPEG→H.264 failed on an old Braswell N3160 (go2rtc issue 464). On an
  N100 with `#hardware` it is cheap.
- go2rtc's native HTTP-MJPEG source has failed on ESP32-CAM endpoints with
  "unknown error" while the `ffmpeg:` wrapper works (go2rtc issue 1746,
  May 2025). WebRTC/MSE/HomeKit consumers cannot play MJPEG, so any live view
  except the raw MJPEG endpoint implies a transcode.
- **Practical:** keep ESP32-CAM at 640x480–800x600 and 5–10 fps, give go2rtc a
  hardware encoder or budget ~0.4 Pi-4 cores per camera, or move to an
  H.264-capable camera.

## Not verified this round

- Coral retail stock and prices in 2026 (only repo archive dates verified).
- AI HAT+ prices and the Jan 2026 "AI HAT+ 2" (Hailo-10H); Frigate lists only
  Hailo-8/8L/8R.
- Scrypted and Agent DVR pricing; Camect cloud dependence.
- ZoneMinder's shipped defaults for `TELEMETRY_DATA` / `CHECK_FOR_UPDATES`
  (check Options → System after install).
- Viseron and Shinobi external-asset and model-download behaviour.
