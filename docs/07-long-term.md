# 07 — What is better long term (5 to 10 year horizon)

Research date: 2026-09-17, second pass. The first pass (docs 02 to 06) answered
"what exists and is it alive". This pass answers "what will still be
maintainable, purchasable and patchable in 2031 to 2036", and what it costs to
exit if a project dies. Evidence tags: **[fetched]** page read directly;
*[snippet]* search-result text only, page blocked; *[inferred]* our inference.
Most vendor and government sites were blocked from the research sandbox this
pass, so a lot rests on snippets. Every section ends with what was not verified.

<!-- VERDICT -->

## 1. Hardware supply: who commits to still making it

Only one platform publishes dated production commitments that cover a 10-year
window. Everything else is an inference from market behaviour.

| Platform | Stated production-until or EOL signal | Confidence |
|---|---|---|
| **Raspberry Pi 4 Model B** (hardware H.264) | "in production until at least **January 2034**" | High, official wording *[snippet]* |
| **Raspberry Pi 5** (no hardware encode) | "until at least **January 2036**" | High *[snippet]* |
| Raspberry Pi Zero 2 W (hardware H.264) | "until at least **January 2030**" (extended from 2028) | High *[snippet]* |
| Compute Module 4 / 5 | Jan 2034 / Jan 2036 (one forum thread notes older CM4 docs said 2031) | Medium-high *[snippet]* |
| Camera Module 3 | "until at least **January 2030**" | High *[snippet]* |
| Espressif ESP32 (original) | 15 years from 2016 → at least 2031 | Medium *[snippet]* |
| Espressif ESP32-S3 / C6 / **P4** | On Espressif's longevity page, "minimum 12 years" for every listed part. Exact years not retrievable (page blocked). P4 ≥ ~2037 is *[inferred]* from the 12-year minimum. | Low for exact years |
| **Rockchip RV1106 / RV1103** | **No lifecycle statement.** Precedent: RV1126 "officially discontinued since early 2025", replaced by RV1126B (SoMs Sep 2025, cameras Apr 2026). That is a **4 to 5 year chip cycle**. Plan a board re-spin around 2028 to 2030. | Medium for the precedent; RV1106 EOL unknown |
| Luckfox (RV1106 boards) | No commitment. Actively shipping: Pico Ultra W PoE kit $41 to $86 by config, Core1106 castellated SoM (Jan 2025). | Medium *[snippet]* |
| **Ingenic T31** | Still in the current lineup (T41, T33, T32Pro, T31, T23); new T31 modules still marketed. No EOL. | Medium |
| Ingenic **T41** | Shipping in new consumer cameras (Wyze Cam v4) **with secure boot, "no known workaround"** short of desoldering the QFN96 SoC and fitting an unfused one **[fetched]** (Thingino discussion 470). | High |
| **HiSilicon Hi3516** family | No EOL statement. Market share ~60% (2018) → 3.9% (2021) after the Sep 2020 TSMC cut-off; returned in 2023 on SMIC silicon; stays sanction-exposed. OpenIPC now also covers newer Hi3516CV610 and Hi3516DV500. EV300 still listed at JLCPCB. | Medium *[snippet]* |
| **Goke GK7205V300 / V200** | Listed as current on goke.com; GK7205V300 + IMX335 modules still sold. No lifecycle statement. | Low-medium *[snippet]* |
| **SigmaStar SSC338Q / SSC30KQ** | **Volume leader among reflash-friendly SoCs**: ~37% of high-end consumer camera SoCs in 2025 (Fullhan 25%, Ingenic 20%); Dahua's post-HiSilicon supplier; OpenIPC U-Boot updates Jan 2026; **ships from the factory with OpenIPC** in RunCam WiFiLink and Emax Wyvern Link FPV products. | Medium-high *[snippet]* |
| Sophgo / CVITEK SG2002 / SG2000 (RISC-V) | Active (Milk-V Duo, Seeed reCamera); mainline Linux devicetree patches submitted Jun to Jul 2026. No lifecycle statement. Sanction exposure not checked. | Medium |

**What this means:**

- **Pi 4 is the odd winner.** The boards with the long dates *and* a hardware
  H.264 encoder are the older ones. A Pi 4 camera node bought in 2026 has a
  guaranteed like-for-like replacement until 2034 and the Foundation commits to
  OS support for all models regardless of age *[snippet]*.
- **Rockchip is a 5-year bet, not a 10-year one.** Good momentum (OpenIPC,
  PoE kit, SoM) but the RV1126 precedent says the chip will be replaced, and
  nothing guarantees the replacement gets OpenIPC support.
- **SigmaStar is the safer reflash target than HiSilicon/Goke** on supply
  grounds. It has the market share, it has a factory-open precedent (FPV), and
  it is not under sanctions. OpenIPC's SigmaStar tier is "MVP" not "done", so
  check the specific SoC (SSC338Q, SSC30KQ) before buying.
- **Ingenic consumer cameras are buy-and-hold.** The Wyze Cam v3 is
  discontinued ("won't be coming back" *[snippet]*). Every T41-generation
  camera is fused. The cheapest live Thingino target is the Cinnado D1 at
  under $15. Buy the T31 stock you want now; there will be no new supply of
  flashable Ingenic consumer cameras.

## 2. Regulation 2026 to 2030: what changes on the shelf

Nothing here stops a home buyer building local CCTV anywhere. It reshapes what
is sold and pushes vendors toward auto-update mechanisms that phone home.

| Law / action | Where | Dates | Effect on you |
|---|---|---|---|
| **FCC Covered List, DA 26-635** | US | Effective **2026-07-16**, no phase-in | New Hikvision/Dahua (and Huawei/ZTE/Hytera) gear may no longer be imported or marketed for "covered purposes". Installed units may keep running. Expect the brand-name and Dahua-OEM (Amcrest, Lorex etc.) shelf to thin from H2 2026. White-label HiSilicon cameras are not named; a claim that the order reaches "devices containing covered components" is **unverified**. |
| NDAA §889 | US federal | 2019/2020 | Federal procurement only. Does not apply to consumers. |
| FCC consumer-router action | US | 2026-03-23, revised 2026-07-28 | Affects the router you put in front of the NVR. TP-Link is **not** on the Covered List; the Commerce ban is still only proposed. |
| US Cyber Trust Mark | US | Launched Jan 2025, paused Jun 2025, administrator withdrew 2025-12-19, new applications opened 2026-08-11 | Not operating. Irrelevant for a local-only build. |
| Canada Hikvision order | Canada | 2025-06-27 | Hikvision Canada wound up. Private use not illegal; no sales or support channel. |
| **EU Cyber Resilience Act** (Reg. 2024/2847) | EU | Reporting duties from **2026-09-11**; full application **2027-12-11** | Security cameras are **Annex III Class I "important products"**: conformity assessment, secure by default, no default credentials, free security updates for **at least 5 years**, automatic updates by default where feasible, SBOM. **Does not require local-only operation.** Side effect: it pushes vendors toward update mechanisms that phone home. Open-source projects are "stewards" with lighter duties *[inferred]*. |
| EU RED delegated act 2022/30 | EU | Mandatory **2025-08-01** (EN 18031 harmonised Jan 2025) | Any WiFi/BLE camera placed on the EU market must meet network-protection and privacy requirements. **Wired PoE-only cameras with no radio are outside RED scope** *[inferred]*. One more argument for wired. |
| UK PSTI Act | UK | In force **2024-04-29** | No universal default passwords, published vulnerability contact, published minimum update period. |
| LG webOS LAN scanning | none, design input | Gamers Nexus video 2026-09-06, LG denial 2026-09-09, LG statement 2026-09-12 confirming network scanning as "standard smart TV function"; critics say the update buries the opt-out | Any consumer device on the same L2 segment enumerates your cameras. Isolated VLAN, no mDNS/SSDP reflection, cameras that do not advertise themselves. No lawsuit or regulator action found. |

## 3. Standards trajectory: what interface to bet on

- **RTSP + ONVIF Profile T stays the safe LAN interface through the 2030s**
  *[inferred]*. ONVIF announced 2025-10-09 that Profile S is sunset (no new
  conformance submissions after 2027-03-31); Profile T (2018) is the streaming
  profile; Profile V (cloud, WebRTC, encrypted upload) is a 2026 release
  candidate. WebRTC is arriving as a *cloud and Matter* transport, not a
  replacement for LAN RTSP. go2rtc bridges RTSP → WebRTC (WHEP) already.
- **Matter cameras exist on paper, not on your shelf.** Matter 1.5
  (2025-11-20) added camera and doorbell device types with WebRTC live audio
  and video, local or remote via STUN/TURN; 1.5.1 (~2026-03-31) added
  multi-stream and CMAF uploads. Reality: the first certified camera (Aqara
  G350, Mar 2026) onboards only via SmartThings, which is the only platform
  with shipping camera support as of Jun 2026. Apple/Google/Amazon committed,
  not shipped. Reolink, Eufy, Arlo, Ring, Nest, Blink uncommitted.
- **An open camera can implement Matter.** The connectedhomeip repo has
  `examples/camera-app` for Linux x86_64 and Raspberry Pi ARM64 (GStreamer /
  FFmpeg pipeline, WebRTC via libdatachannel) **[fetched]**, Apache-2.0. Home
  Assistant's Matter server moved to matter.js (Jun 2026) with experimental
  camera live view. Cross-ecosystem interop still needs CSA certification and
  a device attestation certificate *[inferred]*, so this is a 2028+ option for
  DIY, and only interesting if you want Matter controllers, not Frigate.

## 7. Closed professional cameras: the honest alternative

If longevity of *firmware support* matters more to you than auditability,
these beat every open route on support years. They fail the "unable, not
configured" test in `01-threat-model.md`, and the price is 3 to 8× a reflashed
consumer camera. Prices are single-retailer snapshots *[snippet]*.

| Vendor | Firmware support | Local-only | Entry PoE IR bullet |
|---|---|---|---|
| **Axis** | Active track + LTS every 2 years, each LTS ~5 years; patches typically 5 years after discontinuation. AXIS OS 12 → LTS 2026. ACAP native SDK examples are Apache-2.0 **[fetched]**. | No account needed (ONVIF/RTSP/VAPIX) | AXIS M2035-LE ~$379 to $540 |
| **Hanwha Vision** | Published "Long-Term Firmware Support Policy V3.0" (2024-07-12): security updates up to **5 years after EOL** *[snippet]* | ONVIF/RTSP | ANO-L6012R ~$161 |
| Mobotix | No published support length | Local | premium, not verified |
| Ubiquiti UniFi Protect | No published policy; older G3 now "vintage" (security-only) | Yes: local admin created with WAN disconnected, auto-updates off, no ui.com sign-in needed. Requires a UniFi console. | G5 Bullet $129 |
| Reolink | No published years; "Works with Home Assistant" Platinum partner since 2025-04 (commits to unquantified long-term support) | Disable UID/P2P under Network → Advanced. **Battery models cannot disable UID.** Relays p2p0 to p2p15.reolink.com UDP 9999. | RLC-520A ~$45, RLC-810A ~$55 |
| Amcrest (Dahua OEM) | No published years | P2P disable exists; **forum reports of P2P re-enabling after reboot and outbound traffic persisting** | ~$50 |
| Dahua / Hikvision | Dahua: security updates ≥2 years after first shipment | P2P / Hik-Connect disable | **US import and marketing banned from 2026-07-16**; Canada wound up |

The Amcrest and Reolink rows are the "configured, not unable" failure in one
line: a P2P toggle that can flip itself back is exactly the behaviour open
firmware removes.

## 8. "Open by design" products since 2024

- **Seeed reCamera** (Sophgo SG2002, 1 TOPS): fully open hardware, KiCad
  sources published **[fetched]**, $35 to $55 at launch *[snippet]*. Not
  weatherproof, no IR, no PoE out of the box. A development platform, not a
  camera you mount outside yet.
- **RunCam WiFiLink / WiFiLink 2, Emax Wyvern Link**: commercial products
  shipping **with OpenIPC from the factory** on SigmaStar SSC30KQ/SSC338Q. FPV
  video links, not CCTV, but proof that a factory-open supply chain exists and
  that SigmaStar is where OpenIPC's commercial money is.
- **Luckfox Pico Ultra / Ultra W PoE kit**: the closest thing to an
  open-firmware PoE camera board you can buy today.
- **No CCTV camera ships with OpenIPC or Thingino pre-installed at retail as
  of 2026** *[snippet]*.
- **No Pi-based commercial security camera** exists; only modules and kits
  (Waveshare RPi IR-CUT OV5647, Pimoroni night-vision module).
- **"Works with Home Assistant"** has one camera partner, Reolink (closed
  firmware, local-capable). There is no "Frigate-certified" programme.

<!-- SECTIONS 4 5 6 9 PENDING -->

## Not verified this pass (sections 1, 2, 3, 7, 8)

- Espressif per-series years for S3/C6/P4 (page blocked; only "minimum 12
  years" and ESP32 ≥ 2031 confirmed).
- Raspberry Pi dates are official wording read from snippets; CM4 has a
  2031-vs-2034 discrepancy between documents.
- Whether the FCC July 2026 order reaches white-label devices containing
  HiSilicon parts.
- HiSilicon EV300 manufacturing status and Goke lifecycle: only "still
  listed / still sold" signals.
- Rockchip RV1106 EOL is inferred from the RV1126 precedent.
- Whether any shipping Matter camera can be commissioned with zero vendor
  cloud (Aqara G350 needs SmartThings).
- Telemetry documentation for Axis, Hanwha, Ubiquiti, Reolink, Amcrest; Mobotix
  support length.
- All prices.
