# 07 — What is better long term (5 to 10 year horizon)

Research date: 2026-09-17, second pass. The first pass (docs 02 to 06) answered
"what exists and is it alive". This pass answers "what will still be
maintainable, purchasable and patchable in 2031 to 2036", and what it costs to
exit if a project dies. Evidence tags: **[fetched]** page read directly;
*[snippet]* search-result text only, page blocked; *[inferred]* our inference.
Most vendor and government sites were blocked from the research sandbox this
pass, so a lot rests on snippets. Every section ends with what was not verified.

## Verdict: separate the durable core from the consumable edge

The two things you want, **auditable firmware** and **10-year supply**, do not
live on the same camera today. The most open camera software (Thingino) runs on
a hardware generation whose supply is ending. The most durable open camera node
(Raspberry Pi 4) costs the most labour. The cheapest wired open route (OpenIPC
bullets) has the weakest software governance and permanent vendor blobs. The
best-supported firmware (Axis) is closed. So the long-term design is not "pick
the best camera". It is:

1. **Make the NVR and the network the durable core, and build it first.**
   Frigate on an Intel-iGPU x86 box, a camera VLAN with no WAN route, WireGuard
   on your own router. This layer has the best governance in the whole stack
   (Frigate, Inc., three people with merge rights; Open Home Foundation behind
   Home Assistant), the widest hardware abstraction (OpenVINO on an Intel iGPU
   has eleven years of backward support and needs no vendor kernel), standard
   recordings (plain MP4 segments readable without the software), and an
   interface to cameras, RTSP + ONVIF Profile T, that will outlive every camera
   you buy. Budget one careful Frigate upgrade a year. Archive the container
   image and Frigate's first-start downloads on your own disk. Keep MediaMTX in
   reserve as the restreamer, because go2rtc is one person and has had no code
   change since March 2026.

2. **Treat cameras as consumables on a 3-to-5-year cycle and choose them by
   exit cost, not by longevity.** Any camera that speaks RTSP/ONVIF and cannot
   phone home is replaceable without touching the core. Under that lens, ranked
   for your stated priority (no manufacturer code path to a cloud):

   | Rank | Camera route | Why | Horizon |
   |---|---|---|---|
   | 1 | **Raspberry Pi 4 (or CM4) + switchable IR-cut module + MediaMTX**, for the positions that matter most | The only DIY route where nothing is a blob you cannot rebuild: libcamera's Pi ISP algorithms are BSD-2, MediaMTX and Debian are funded and active, and the board is in production until **January 2034**. Costs ~€145 and 4 W per node plus the most labour; use read-only root; buy spare IR-cut modules. | **10 years** |
   | 2 | **OpenIPC on SigmaStar (SSC338Q/SSC30KQ) PoE bullets**, for the rest of the outdoor positions; Goke/HiSilicon if that is what you can get | Cheapest weatherproof, IR-cut, PoE route with an open userland. Accept a frozen 4.9 kernel and ISP blobs; on a no-WAN VLAN that is tolerable because no patches were ever coming. **SigmaStar over HiSilicon on supply grounds**: 37% market share, not sanctioned, already ships OpenIPC from the factory in FPV products. Mirror the OpenIPC release assets locally; Majestic is a binary fetched from S3 without a hash. Expect to re-qualify a model every 2 to 3 years. | **5 years per model** |
   | 3 | **USB UVC IR camera on the NVR** for the one or two positions within 5 m of it | No firmware, no network stack, no maintenance. Nothing to exit from. | 10 years |
   | 4 | **Thingino on Wyze Cam v3** (T31 revision) for cheap indoor cameras | Best firmware openness and patch currency of any camera route, but the flashable hardware supply is closing (v4 is fused; the Tapo C500 bootloader stopped being interruptible in Jan 2026). Buy the units you want now, plus spares. | **3 to 5 years** |
   | 5 | **Luckfox Pico Ultra** as a tinkering node | Cheapest PoE node with hardware H.264, but vendor kernel 5.10 (LTS ends Dec 2026) with blobs, a 4-to-5-year Rockchip chip cycle, and no IR-cut module. | 5 years |
   | 6 | **ESP32-P4** | Hardware encoder is now open source and PoE boards exist, but the only active firmware is a one-person beta, WiFi-only, still finishing RTP packetisation. | **Re-evaluate 2028** |
   | 7 | **Axis or Hanwha closed cameras on the VLAN** | Only if you decide that five years of vendor patches after discontinuation is worth more to you than auditability. Three times the price, lowest labour, and it fails the "unable, not configured" test in `01-threat-model.md`. | 10 years, closed |

3. **The 10-year bill for six cameras** (labour excluded, €0.30/kWh, estimates):
   Thingino WiFi ≈ €870, Luckfox ≈ €1,160, USB ≈ €1,250, OpenIPC bullets ≈
   €1,330, Pi 4 nodes ≈ €2,130, Axis ≈ €4,120. At €30/h of your time the Pi
   and Axis routes converge (≈€5,700 vs ≈€5,000) and OpenIPC bullets stay
   cheapest at ≈€3,700. Your hourly rate, not the hardware, decides between
   Pi and Axis.

4. **What to do now:** buy the NVR box and switch first, put every camera you
   already own on the isolated VLAN, then add one Pi 4 node at the position
   that matters most and OpenIPC SigmaStar bullets elsewhere. Keep local
   mirrors of every firmware image, tree and model file. Put "check ESP32-P4
   and Rockchip mainline" in the calendar for 2028.


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

## 4. Camera firmware: who maintains it, and what breaks if they stop

Ranking from this pass, for a 5 to 10 year fully local horizon:
**(1) Thingino on Ingenic T31-class hardware, (2) OpenIPC, (3) buy Rockchip and
wait for mainline.** The surprise is that the project with the bigger
organisation and commercial customers ranks second on the things that matter
for a decade: source availability of every layer you might have to fork,
userland patch currency, and whether an image can be rebuilt offline.

### Governance

| | OpenIPC | Thingino |
|---|---|---|
| Repo | `OpenIPC/firmware`, MIT, 3,452 commits, 2.2k stars, 475 forks | `themactep/thingino-firmware`, MIT, 8,552 commits, 2.1k stars, 310 forks. Personal repo, plus a `thingino` org for tooling. |
| Commit / merge rights | 24 public org members (founder flyrouter = Igor Zalatov; widgetii = Dmitry Ilyin; cronyx, mikecarr, nekromant, MarioFPVdev...) | ≥3: themactep (owner, Paul Philippov), gtxaspec (collaborator, author of wz_mini_hacks and Raptor), Lu-Fi (merged own PR). |
| Who actually commits | **Aug–Sep 2024:** viktorxda 10, cronyx 7, flyrouter 4. **Jul–Sep 2025:** viktorxda 11, flyrouter 5. **Aug 2025–Jan 2026: ~29 commits in 5.5 months.** **May 2026 (10 days):** widgetii 27, "claude" co-author 17. **Sep 7–16 2026:** widgetii 24, `openipc-ai` 8. viktorxda's last commit 2026-07-23. | **Sep 2024:** themactep 28/30. **Oct 2025:** 26/28. **Jan–Feb 2026:** 27/28. **May 2026:** themactep 17, gtxaspec 6, Eric 2, m4mmon 1. **Sep 13–16 2026:** gtxaspec 13, themactep ~14, Lu-Fi 5, WLTB-Gino 1. |
| Velocity trend | Near-dormant late 2025 (~5 commits/month) → 100+/month in 2026, almost all from one maintainer with AI co-authorship (PRs credit "Claude Opus 4.6"; `openipc-ai` org created 2026-08-19; `CLAUDE.md` in repo). | Steady weekly tagged releases (2026-09-10, 09-14, 09-15). The 2026 change: gtxaspec became co-equal and a second tier appeared (wltechblog installers, Lu-Fi, m4mmon, matteius of OpenSensor for the ISP). |
| Funding / entity | Open Collective (amounts blocked), paid commercial support offered in README, own hardware store (AIO boards for FPV). **Commercial customers: RunCam WiFiLink/WiFiLink 2 and Emax Wyvern Link ship OpenIPC.** No legal entity identified. | GitHub Sponsors only: themactep 2 current / 14 past; gtxaspec 1 current / 8 past. No entity, no paid support, no factory product. Contributions page asks for hardware donations. |
| Roadmap | Published, **entirely FPV hardware** (Thinker, Bonnet, Goggles, Evolution AIO). No CCTV, kernel or security items. | None published. Branch policy instead: `ciao` stable, `master` experimental (Raptor, mainline U-Boot, "UART access highly recommended"). |
| Security policy | No SECURITY.md. One advisory: GHSA-fjf7-9x3v-6mj6, High, 2026-09-06, sysupgrade fetched firmware without TLS certificate verification. | No SECURITY.md, no advisories. Fixes appear in release notes (2026-09-14: API key bypass for loopback fixed, RTSP Digest auth added). FAQ: "does not collect any data, metrics, or telemetry". |
| Userland currency | **Buildroot pinned to 2024.02.10** with no download verification; overrides **dropbear 2022.82** and **mbedTLS 2.25.0** with no patches. | **Buildroot 2026.08** (release 2026-09-10); own mbedTLS/OpenSSL packages. |
| Update mechanism | `sysupgrade` from GitHub releases or an openipc.github.io manifest, **md5 only, no signature**; U-Boot boot-count failsafe added Sep 2026; no rollback. | `sysupgrade` self-updates from the GitHub `stable` branch then flashes; no rollback documented; wiki does not describe verification. |
| **Bus factor verdict** | Org is broad, but firmware work is effectively **bus factor 1 (widgetii)** plus the founder at low volume. The 2024–25 core (viktorxda, cronyx) has receded. Majestic's developers are not publicly identifiable, so the closed streamer is a **second single point of failure**. | **Bus factor 2** (was 1 until 2026). No divergent forks found. |

Neither project documents VLAN isolation. Neither signs images. Both run update
scripts fetched from GitHub as root. Vendor kernels (3.10.14, 4.4.94, 4.9.x)
receive no upstream CVE fixes in either project.

### Kernel and blob trajectory per SoC

| SoC | Kernel today | Mainline (torvalds/master, 2026-09-17) | Blobs | 5-year outlook |
|---|---|---|---|---|
| **Ingenic T31** | 3.10.14 vendor in both projects. Thingino tree also carries `t31.generic.config` for **6.11 and 7.1-rc1** (experimental, dev-only?). | No T-series in `arch/mips/boot/dts/ingenic`. Ingenic-community/linux explicitly excludes T-series ("controlled by a subsidiary with different policies"). But **open-tx-isp is "compatibility-tested on mainline Linux 7.1"** **[fetched]**. | ISP: open-tx-isp (GPL-3) replaces tx-isp.ko, "near-OEM daylight parity", gaps in night/IR, WDR. Userland: OpenIMP replaces libimp. Still proprietary: libalog, libsysutils, audio processing, OEM tuning tables, `*.o_shipped` encoder/audio objects in thingino/ingenic-sdk. In Thingino's build it is opt-in and wired only to the vendor kernels. **All 34 recent open-tx-isp commits by one developer (matteius, OpenSensor Engineering LLC, Maine). 19 stars.** | **Most credible blob-light, mainline-kernel path of any camera SoC.** Hinges on one developer and on a hardware generation no longer sold new in flashable form. |
| Ingenic T40 / T41 | 4.4.94 vendor | None | open-tx-isp device-tested on T40/T41; OpenIMP T41 in bring-up. | Irrelevant for consumers: secure boot on "all the newer Ingenic based devices". |
| **HiSilicon Hi3516EV300** | OpenIPC `lite`: 4.9.37. OpenIPC **`neo`: Linux 7.0** + OpenHisilicon SDK **[fetched]**. | No hi3516* in mainline dts. OpenIPC's `upstream-patches` branch holds DT/CRG/mach patches "intended for mainline"; no LKML submission found. | OpenHisilicon: GPL-3 modules, but README admits "the source code for most of these modules is not provided": it **relinks vendor `.o` blobs** via an OSAL layer. ISP 3A libraries, sensor libs and firmware stay proprietary. Neo 6.6: "boots to login... majestic" in QEMU (Apr 2026); 7.0-rc6: "Real hardware: pending". README table says "Production". | **Best modern-kernel story of any CCTV SoC, worst blob story.** Entirely dependent on widgetii. Hardware abundant and cheap. |
| Goke GK7205V200/V300 | 4.9.37, actively patched Sep 2026 (pstore, failsafe rescue) | None | Same as HiSilicon V4. No neo build seen. | Follows EV300; no independent path. |
| **SigmaStar SSC33x (Infinity6/6E)** | 4.9.84 vendor | `arch/arm/boot/dts/sigmastar` has infinity, infinity2m, infinity3, mercury5. **No infinity6.** | Vendor MI/ISP blobs; no open reimplementation. Majestic-dependent. | **Static: works, unpatchable kernel, no visible path.** Positive: the IPL boots a custom U-Boot from SD, so the hardware is not locked. |
| **Rockchip RV1106 / RV1103** | OpenIPC 5.10.160 vendor (merged Sep 2026, Luckfox Pico Max) | **RV1103B dts merged 2026-03-24** (Onion Omega4). RV1106/RV1103 + Luckfox Pico Mini B series by Simon Glass: v1 2026-07-06, v2 07-14, v3 07-29 (clocks, GRF, UART, SD, SPI-NOR, GPIO, pinctrl, USB2 PHY). Not in master as of 2026-08-02; merge status unverified *[snippet]*. | OpenIPC installs **prebuilt `rockchip-osdrv-rv11xx` libs and 5.10 kmods** (package has empty SITE/VERSION, license mislabelled MIT). Mainline `rkisp2` (RK3588 ISP 3.0, v3 Aug 2026) does not mention RV1106's ISP32. **No mainline H.264 encoder driver**: Hantro H1 stateless uAPI still RFC since 2023. | **Platform in mainline within 1 to 2 years; camera pipeline not** without a Collabora-scale effort nobody has announced. A dev-board bet, not a camera bet. |

### Secure boot and supply

- gtxaspec, 2025-01-29: secure boot "introduced with the Pan V3, and found on
  the Roku variants, as well as all the newer ingenic based devices... No known
  workarounds since they validate the bootloader signature" **[fetched]**.
  2025-07-03: Wyze v4 (T41) needs a hot-air SoC swap.
- **Tapo C500 (T23):** units with a bootloader dated 2026-01-06 no longer offer
  the autoboot prompt (Sep 2024 units were interruptible). Issue open
  2026-09-12, no maintainer reply. Whether it is signature-enforced is not
  established. Treat as the leading edge of the lock-down spreading to T23.
- **SigmaStar is not locked** (IPL boots custom U-Boot from SD). Xiongmai has
  had a bootloader password since ~2021, bypassable by downgrade.
- **Wyze Cam v3 conflict:** the product page says "no longer available and
  won't be coming back" *[snippet]* while Wyze's support page was updated
  Mar 2026 and firmware betas continued through Sep 2026. Read it as: firmware
  still maintained, new retail supply ended, four different SoC/WiFi
  combinations exist in the wild. Buy remaining or secondhand stock
  deliberately.
- Nothing ships Thingino from the factory. OpenIPC ships from the factory only
  in FPV products and its own AIO boards. OpenIPC's board-manufacturer wiki
  lists ~50 vendors with no compatibility notes.

### Exit cost

| If this dies | Deployed cameras | Rebuilding images | NVR side |
|---|---|---|---|
| **OpenIPC** | Keep running (no licence server found; not verified). | **Majestic is fetched at build time from `openipc.s3-eu-west-1.amazonaws.com` as an unversioned tarball, `LICENSE = PROPRIETARY`, no hash** **[fetched]**. If the bucket disappears, nobody can rebuild an image with Majestic. Mitigations: 480 prebuilt assets in the `latest` release, a self-hosted mirror at git.openipc.ru *[snippet]*, and Divinus (MIT, 616 commits) for RTSP/fMP4/JPEG/audio/OSD but **no ONVIF write, PTZ or motion detection**. | Unaffected: RTSP + ONVIF via `onvif-simple-server`. |
| **Thingino** | Keep running. | **A git clone rebuilds offline indefinitely**: all code GPL/MIT, Ingenic blobs redistributed inside the tree, vendor kernels in project forks. | Unaffected: RTSP + `thingino-onvif`. |
| Migration OpenIPC → Thingino (same Ingenic SoC) | Documented: flash gtxaspec `u-boot-ingenic` to mtd0, wipe env, `autoupdate-full.bin`. "Overwrites everything in the flash chip." | | Reverse direction undocumented. Different U-Boot, env and partition layouts; not hot-swappable. |

**Practical hedge:** choose NOR-flash Ingenic T31 cameras that both projects
support, keep every camera on an isolated VLAN (neither project will do that
for you), and let the NVR depend only on RTSP and ONVIF so a dead firmware
project leaves you with frozen appliances rather than bricks. On a VLAN a
frozen HiSilicon or SigmaStar camera is tolerable, because no kernel patches
were ever coming for it anyway.

### Not verified (section 4)

- Open Collective totals for OpenIPC; Telegram community sizes (3,120 FPV
  members is a telemetr.io snippet).
- Merge status of the RV1106/RV1103 kernel series after v3 (2026-07-29).
- Whether rkisp2 will cover RV1106's ISP32; 2026 status of Hantro/VEPU encoder
  drivers.
- Real-hardware video on OpenIPC `hi3516ev300_neo` 7.0 after April 2026.
- Whether a T31 Thingino image can be built with zero Ingenic objects; whether
  the 6.11/7.1 configs are user-selectable.
- Thingino sysupgrade verification details; Majestic phone-home or licence
  behaviour; Buildroot 2024.02 EOL date.
- Tapo C500 signature enforcement; lock status of SigmaStar/Reolink/Xiaomi
  consumer cameras.

## 5. NVR software and detection hardware: who maintains it, what it costs to upgrade

**Safest 10-year NVR stack: Frigate as the recording and detection core on an
Intel-iGPU x86 box, MediaMTX kept in reserve as the restreamer if go2rtc
stalls, and Home Assistant only loosely coupled over MQTT.** Frigate is the
only project in the set with more than one person holding merge rights, a
legal entity with a revenue product behind it, and a detector abstraction wide
enough that the software outlives any single accelerator. Its cost is a
breaking release every 6 to 7 months.

**Safest detection hardware for 5+ years: an Intel iGPU driven by OpenVINO.**
Eleven years of backward support so far, mainline kernel driver, and the same
silicon decodes the video.

### Governance

Measurement note: GitHub's contributors endpoint returned 400, so counts come
from the last 20 to 24 commits per repo and per author (Atom feeds). Treat
">10 commits/year" as a lower bound.

| Project | Who actually commits (measured) | Entity / funding | Velocity | Bus factor | Verdict |
|---|---|---|---|---|---|
| **Frigate** (MIT) | Last 20 dev commits (14–16 Sep 2026): hawkeye217 13, dependabot 6, NickM-27 1. NickM-27: 21 commits in 22 days. hawkeye217 (labelled Collaborator) approves and NickM-27 merges; Blake Blackshear owns the repo and cuts releases (20 commits in 11 months, mostly release merges) **[fetched]**. | **Frigate, Inc.** (was Frigate LLC; "llc to inc" PR merged 2026-01-01). Frigate+ $50/yr. GitHub Sponsors: 104 sponsors. LinkedIn snippets suggest the maintainers have day jobs *[snippet]*, so no evidence Frigate+ funds full-time staff. | Up: 0.16 Aug 2025 → 0.17 Feb 2026 → 0.18 Sep 2026; 0.19 in dev. | **2 to 3** | Best in class for this niche. Single-owner repo and company is the residual risk. |
| **go2rtc** (MIT) | Last 20 master commits span Feb–Jul 2026: AlexxIT 14, three others. **Last code change 2026-03-17. No release since v1.9.14 (2026-01-19), 8 months. 193 open PRs vs 280 closed**, dozens opened mid-Sep 2026 unmerged. AlexxIT is alive and active elsewhere (18 SonoffLAN commits Sep 2026) and still closes go2rtc issues **[fetched]**. | No entity, no sponsorship statement, no co-maintainer. Fork status not checked. | **Down** in 2026 | **1** | Fine while bundled and pinned inside Frigate. Do not build on it standalone for 10 years. |
| **MediaMTX** (MIT) | Last 20 commits: dependabot 11, bot 5, aler9 4; aler9 alone: 20 commits in 9 days. Org members: aler9 + a bot **[fetched]**. | No FUNDING.yml, no Sponsors profile, no commercial offering found. | Steady, high (v1.21.0 Sep 2026) | **1** | Excellent code and cadence, zero redundancy. Standard fMP4/TS recordings limit lock-in. |
| **Viseron** (MIT) | Last 20 commits: roflcoopter 20/20 **[fetched]**. Detector components: darknet, edgetpu, hailo, yolo, codeprojectai, deepstack. **No openvino, onnx or rknn.** | Sponsors + Buy Me a Coffee. | Steady | **1** | On an Intel box it has no iGPU detection path. Not a 10-year primary. |
| **ZoneMinder** (GPL-2.0) | Last 20 commits: connortechnology 15, SteveGilvarry 3; several carry "Claude Opus 5" co-author trailers **[fetched]**. | No foundation; PayPal, Patreon, Sponsors. | Up in 2026 (1.38.0 Feb 2026 after 1.36.0 in May 2021) | **1 (+1)** | 20+ year project with distro packaging and 5-year support branches, carried by one person. |
| **Moonfire NVR** (GPL-3) | scottlamb 19/20, bursty **[fetched]**. README: "pre-1.0... configuration and storage formats may change", "Help wanted". | None. | Low | **1** | Not a 10-year bet by its own statement. |
| **Home Assistant** (Apache-2.0) | Thousands of contributors. | **Open Home Foundation** (Switzerland), ~70 staff in 2026; commercial partners give a majority of licensed-product profit to the foundation *[snippet]*. | Very high | **>10** | Strongest governance, highest churn. Integrate via MQTT so the CCTV core survives HA changes. |

### Upgrade burden

| System | Cadence | What breaks |
|---|---|---|
| **Frigate** | One breaking major every **6 to 7 months** (0.13 Jan 2024, 0.14 Aug 2024, 0.15 Feb 2025, 0.16 Aug 2025, 0.17 Feb 2026, 0.18 Sep 2026), 5 to 9 breaking items each, mostly auto-migrated config. | Hardware drops every 1 to 2 releases: ARM 32-bit (0.13), **Intel Neural Compute Stick (0.14)**, standalone TensorRT and ROCm detectors and Jetpack 4/5 (0.16), **GTX 900 (0.17)**, DeGirum (0.18), DeepStack (0.19). 0.14 could not migrate existing events. 0.18 needs kernel ≥6.5 for Intel stats, FFmpeg 8 default, JinaV2 GPU users must reindex. |
| **ZoneMinder** | ≈2 majors per decade. 1.36 → 1.38: ~79 schema updates applied **automatically**. SECURITY.md: 1.36.x still gets "best-effort security fixes" after **~5 years** (1.36.5 Jun 2021 → 1.36.38 Feb 2026) **[fetched]**. | Some renamed parameters; back up the DB. |
| **Home Assistant** | Monthly. 2026.9: **8** backward-incompatible entries; 2026.3: **10** **[fetched]**. ≈100 breaking entries per year. | Keep it out of the CCTV critical path. |

Lowest maintenance over 10 years: ZoneMinder < Frigate (≈15 breaking releases
per decade, plan one careful upgrade per year with a config and DB backup) <
Home Assistant. Moonfire promises nothing.

### Detection hardware for 5+ years

| Hardware | Kernel driver | Vendor status 2026 | Frigate 0.18 | Verdict |
|---|---|---|---|---|
| **Intel iGPU** (Skylake → Core Ultra 3, incl. N100/N150) | mainline i915/xe | OpenVINO 2026.x supports "6th to 14th generation Intel Core", Core Ultra 1/2/3, all HD/UHD/Iris Xe iGPUs **[fetched]**. 4 to 5 releases a year. Policy promises an annual LTS with 2 years of security fixes, but **the last release labelled LTS is 2023.3** **[fetched]**. **2026.3 made AVX2 mandatory for the CPU plugin** (drops SSE-only Gemini/Jasper Lake Celerons for CPU inference). | Official | **Safest.** No vendor kernel, doubles as the video decoder. |
| Intel NPU (Core Ultra) | mainline `intel_vpu` (Meteor Lake+) **[fetched]** | Firmware, driver and compiler versions must match. Not on N-series. | Official | Good but version-coupled. |
| **Hailo-8 / 8L** (Pi 5 AI HAT+, M.2) | **Out-of-tree GPL `hailo_pci`; nothing in mainline `drivers/accel`** **[fetched]** | HailoRT MIT, still releasing for Hailo-8 (runtime v4.24.0 Jun 2026, driver 2026-09-15) but **Hailo-8 lives on a maintenance branch**; master is Hailo-10/15 only. Company: valuation halved, SPAC collapsed, ~50% layoffs, then **Microchip signed a definitive agreement to acquire Hailo on 2026-07-24, closing expected by 2026-09-30** *[snippet]*. | Official; downloads HailoRT and a model at first start (needs internet once). | **Acceptable with caveats.** Microchip is a long-lifecycle embedded vendor, which likely helps supply. Software stays vendor-bound with a proprietary compiler. Cache the runtime, buy a spare. |
| Hailo-10H | separate driver v5.x; HAOS PR still unmerged (firmware 108 MB exceeds partition) | | Not in 0.18 docs | Wait. |
| **Google Coral** | out-of-tree gasket/apex | Upstream archived. | "no longer recommended" | Avoid for new builds. |
| **Rockchip RK3588/3576 NPU** | Mainline `rocket` in `drivers/accel` since **Linux 6.18** (RK3588 only; kernel "just powers the hardware on and off", everything else in Mesa Teflon) **[fetched]** | Frigate requires the **vendor BSP kernel 5.10 or 6.1** with `rknpu` ≥0.9.2 and RKNN-Toolkit2; it does not use `rocket`. | Official (`-rk` image) | **Avoid** for a 10-year build: vendor-kernel dependency. |
| Nvidia dGPU | proprietary/open modules | GTX 900 dropped 0.17 | Official | Works, highest power and cost. |
| AMD dGPU (ROCm) | mainline amdgpu | ROCm 7.2, RDNA4 | Official | Second tier. |
| AMD Ryzen AI NPU | mainline `amdxdna` since 6.14 | ONNX Runtime: "Ryzen AI Linux support is not enabled" *[snippet]* | none | Avoid. |
| MemryX, AXERA, Synaptics, Jetson | vendor SDKs | | "community supported" | Avoid. |

**Does ONNX make hardware swappable?** Partly. Frigate's `onnx` detector
auto-selects CPU, OpenVINO, CUDA/TensorRT or ROCm, so one YOLO model moves
between Intel, Nvidia and AMD with a config change. The NPU family (Hailo
`.hef`, Coral TFLite, RKNN, AXERA, MemryX) each need a vendor compiler.

### NVR host and 10-year running cost

| Host | Longevity signal | Idle / avg watts *[snippet]* | 10-year electricity at **€0.29/kWh** (Eurostat EU average H2 2025) |
|---|---|---|---|
| **Intel N100 → N150/N250** mini PC | Twin Lake is a clock-bumped refresh of the same silicon; N150 replaced N100 in 2026 mini PCs. Consumer boxes carry no lifecycle promise (treat as 3 to 5 year disposable with like-for-like always available); industrial boards on the same chips are sold as "10-year long-life". | 9 to 14 W idle; ~15 W avg assumed | **≈€381** (€305–508) |
| Used ThinkCentre Tiny (8th gen) | Abundant used, standard parts, inside OpenVINO's window, HEVC 10-bit decode, no AV1. | 11 to 14 W idle; ~22 W avg | **≈€559** (€457–762) |
| Raspberry Pi 5 + Hailo | "until at least January 2036" (product page) vs "January 2038" (longevity article), conflicting snippets. **No H.264 hardware decode**; Frigate users report 50 to 85% CPU decoding H.264 in software. Pi 5 NVR only with H.265 cameras. | ~2.7 W idle; ~8 W avg | **≈€203** (€152–254) |

The Pi's decade of savings (€180 to 350) is roughly the price of the Hailo HAT
it needs, and it costs H.264 flexibility. The N100-class box is the balanced
choice. One watt continuous is €25.40 per decade at this rate.

### Data and format longevity

| System | Recordings on disk | Readable without the software? |
|---|---|---|
| **Frigate** | ~10 s MP4 segments written without re-encoding, `YYYY-MM-DD/HH/<camera>/MM.SS.mp4` **[fetched]**; SQLite metadata | **Yes.** Concatenate with ffmpeg. Events and review items are Frigate-specific and were not migrated even between 0.13 and 0.14. |
| **MediaMTX** | fMP4 or MPEG-TS segments, built-in playback server | **Yes.** |
| **ZoneMinder** | JPEG frames and/or MP4 (passthrough) per event; MySQL | **Yes** for the media files. |
| **Moonfire** | Raw concatenated samples ("contents of a `mdat` box... not meant to be decoded on their own") + SQLite index **[fetched]** | **No.** Needs the DB to reconstruct MP4. |

**Codecs:** Frigate's presets and go2rtc's codec table cover H.264 and H.265
only; **no AV1 anywhere** **[fetched]**. Only Axis ARTPEC-9 cameras ship AV1
today. Record H.264 for maximum compatibility, or H.265 if the host is a Pi 5
or storage-bound. Ignore AV1 until the NVR side adds presets.

### Fully local, 10 years out

Frigate downloads at first start: HailoRT into `/config/.local`, the Hailo
default model, the RKNN default model (GitHub), AXERA models (HuggingFace).
0.19-dev adds "dynamically install and load detector dependencies". An offline
plan needs the container image, `/config/.local` and model files archived on
your own disk, or you cannot rebuild the NVR without internet in 2033.

### Not verified (section 5)

- 12-month contributor counts (endpoint failed); only commit rates measured.
- Frigate, Inc. jurisdiction; whether anyone works on Frigate full time.
- Frigate 0.13/0.14/0.15 years inferred from version sequence.
- go2rtc forks not checked; "MediaMTX Pro" not found (site blocked).
- Hailo deal terms and whether the acquisition closed; Hailo-8 EOL date.
- Pi 5 production date (2036 vs 2038 conflict).
- All power figures and the Eurostat price are snippet-only; load averages are
  assumptions.
- Intel decode-by-generation table from memory.
- Whether OpenVINO 2025.x still gets security patches; whether the AVX2
  requirement affects the GPU plugin on old Celerons.

## 6. DIY camera routes and the 10-year bill

Six cameras, ten years, **€0.30/kWh** (1 W continuous = €26.28 per decade),
dollars treated as euros for totals. Hardware lifespans and maintenance hours
are **estimates** with the reasoning stated in the per-route notes. Prices are
snippets or estimates.

### Ranking for 10-year viability

| Rank | Route | Initial (6 cams + switch) | Life and main failure modes | Replaceable in 2031? | Maint. h/yr | Patchability | Exit cost if firmware dies | 10-yr power | **10-yr total, labour excluded** |
|---|---|---|---|---|---|---|---|---|---|
| 1 | **Axis-class closed cameras on a VLAN** | ≈$2,790 | 8 to 12 y; IR LEDs, capacitors; industrial sealing | Excellent: AXIS OS ≥5 y after discontinuation, LTS tracks ~5 y | ~3 | Best cadence; **must block phone-home at the firewall** | Low: ONVIF/RTSP keep working offline | 33.6 W → €883 | **≈€4,120** |
| 2 | **Pi 4 + Arducam IR-cut + MediaMTX** | ≈$955 ($45 Pi 4 2 GB + €40 Arducam + $22 PoE HAT + $10 SD + $25 enclosure per node, + $90 switch) | Board 8 to 10 y; **SD 1 to 3 y unless read-only root**; IR-cut solenoid and LEDs 3 to 5 y; DIY enclosure ingress; ribbon connector wear | **Excellent: Pi 4 to 2034-01-01, Zero 2 W to 2030, Pi 5 to 2036** **[fetched, endoflife.date data]**; OV5647 ubiquitous | ~12 (apt and kernel ×6, MediaMTX, SD swaps, occasional libcamera breakage) | **Best of DIY**: Debian security (Trixie to 2028, LTS 2030), Pi kernel fork, MediaMTX all funded and active | Low: any Linux stack works | 33.6 W → €883 | **≈€2,130** |
| 3 | **USB UVC IR cameras on the NVR** | ≈$710 ($55 ELP H.264 IR + $40 extender + $20 enclosure per cam) | 5 to 8 y; IR LEDs, bare-PCB corrosion, extender power sag, connector wear | Good: UVC is generic; but **H.264-over-UVC models vanish silently** (Logitech dropped it from C920/C922 in late 2018 units) | ~3 | Nothing on the camera to patch | **None**: UVC works with any host forever | 15 W → €394 | ≈€1,250 |
| 4 | **OpenIPC on Goke/HiSilicon/SigmaStar PoE bullets** | ≈$330 | 5 to 8 y; IR LED dimming (L70 at 20 to 30k h), gasket ingress, LED-driver electrolytics, **PoE gear in hot outdoor boxes fails** | Model churn high; HiSilicon exited (60% → <4% share); Goke/SigmaStar/Ingenic clones plentiful but **need re-qualification every 2 to 3 years** | ~8 | Userland patched; **kernel 3.10/4.4 + ISP blobs never patched; Majestic closed** | Medium: cameras keep streaming on last firmware; VLAN-isolate | 33.6 W → €883 | **≈€1,330** |
| 5 | **Luckfox Pico Ultra (RV1106) + PoE + SC3336** | ≈$515 | Unknown (product since 2024); eMMC avoids SD wear; **no IR-cut → night colour cast without an add-on** | Uncertain: Luckfox founded 2020; RV1106 mainstream in Chinese IPCs, so other boards likely | ~10 | Vendor **5.10.160 (LTS EOL 2026-12-31)** + rockit/rkaiq/mpp blobs; userland patchable | Medium: stock rkipc RTSP still runs; blobs tie you to 5.10 forever | 16.4 W → €431 | ≈€1,160 |
| 6 | **Thingino on Wyze v3 (WiFi)** | ≈$240 | 3 to 5 y; **SD-card destruction widely reported**, IR-LED aging, WiFi dropouts, indoor plastics outdoors | **Poor**: v4 is T41 with secure boot; other T31/T23 cameras likely still exist | ~8 | Good userland (Thingino active); kernel 3.10 blobs never patched | Medium: revert to vendor firmware = cloud-only, or bin them | 15 W → €394 | ≈€870 (one replacement cycle at year 5) |
| 7 | **ESP32-P4 nodes (when ready)** | ≈$505 | MCU boards 10 y+; no SD; IR LEDs; PoE module | Good chip-wise (Espressif ≥12 y); Waveshare/Olimex stable | ~15 until firmware matures (rebase to a supported ESP-IDF every 30 months) | Small surface; no mature ONVIF auth; IDF fixes only on supported branches | **High today**: no second firmware to fall back to | 17.7 W → €465 | ≈€1,100, **not deliverable in 2026** |

**Labour sensitivity** (estimate at €30/h): adds ≈ Axis €900, USB €900,
Thingino €2,400, OpenIPC €2,400, Luckfox €3,000, Pi €3,600, ESP32-P4 €4,500.
With labour, Pi ≈ €5,700 vs Axis ≈ €5,000 vs OpenIPC bullets ≈ €3,700 vs USB
≈ €2,150 (but USB cannot cover six outdoor positions). At €0/h the Pi route
costs about half of Axis. Your hourly rate decides between rows 1 and 2.

### Raspberry Pi, 10-year view

- **Mainline status is split by generation.** Pi 5's camera path is fully
  upstream: `rp1-cfe` (v6.13) and `pisp_be` (v6.11) in
  `drivers/media/platform/raspberrypi` **[fetched]**. Pi 4 / Zero 2 W: the
  CSI-2 receiver `bcm2835-unicam` is mainline (v6.10), but the **ISP and the
  H.264 encoder are only in the Pi kernel fork** (`bcm2835-codec`,
  `bcm2835-isp`, `vchiq-mmal` in the fork's vc04_services; mainline has only
  `bcm2835-audio`) **[fetched]**. A Pi 4 node with hardware encode depends on
  the Pi fork for its whole life. Mitigation: Pi has kept a 2012 board booting
  the 2025 OS.
- **The camera algorithms are open.** Upstream libcamera carries both Pi
  pipeline handlers and the **BSD-2-Clause IPA** (`src/ipa/rpi/{vc4,pisp}`)
  **[fetched]**. Every other route in this document runs ISP tuning as a vendor
  blob. This is the Pi route's unique long-term property.
- **Production dates** **[fetched from endoflife.date data file]**: Pi 4 B
  2034-01-01, Pi 5 2036-01-01, Zero 2 W 2030-01-01, CM4 2034-01-01, CM5
  2036-01-01. Camera Module 3 to Jan 2030 *[snippet]*.
- **OS support:** Pi OS tracks Debian. Bookworm security to 2026-07-11, LTS to
  2028-06-30; Trixie security to 2028-08-09, LTS to 2030-06-30 **[fetched]**.
  Trixie still supports Pi 1 to Pi 5. After a new release, old releases get
  critical kernel fixes only, and in-place upgrades are unsupported
  *[snippet]*: plan a reflash per node every 2 to 3 years.
- **Pi 5 software encode:** official docs say it "will still easily achieve
  1080p30" **[fetched]**; estimate 0.5 to 1 of 4 cores at 1080p25 and +1 to
  1.5 W. MediaMTX's rpicamera bundle has both `encoder_hardware_h264.c` and
  `encoder_software_h264.cpp` (openh264) **[fetched]**. MediaMTX bundles its
  own libcamera and is "not compatible with cameras that require a custom
  libcamera": fine for OV5647/IMX708, a problem for Arducam sensors that need
  Arducam's fork.
- **Successor with hardware encode:** Eben Upton at Pi 5 launch: "hardware
  encode is a mm² too far... In future we'll have to do something" *[snippet]*.
  Nothing further found. Not verified.
- **No official IR-cut camera.** Third-party only: Waveshare RPi IR-CUT
  (OV5647, LDR-automatic or GPIO versions), Waveshare IMX462 IR-CUT, Arducam
  B0151 OV5647 motorised IR-cut (€39.90, listed for Pi 5/4/3) *[snippets]*.
  Buy one spare per two nodes; the solenoid and LEDs are the weakest link.
- **SD wear:** continuous writes "will wear them out quickly, potentially
  causing corruption after a few months" *[snippet]*. Use overlayfs /
  read-only root via raspi-config, or USB/eMMC boot. Do not record locally on
  the node; the NVR records.
- **Prices after the Dec 2025 rises** *[snippet]*: Pi 5 2 GB $55; Pi 4 4 GB
  $60, 8 GB $85. Pi 4 2 GB assumed $45.

### ESP32-P4, 10-year view

- **Longevity:** "minimum 12 years for all the products listed" *[snippet]*;
  P4 mass production ~2025 implies ≥2037 (estimate).
- **ESP-IDF policy** **[fetched]**: each release supported 30 months (12
  service + 18 maintenance, security fixes only). P4 supported since v5.3.
  Healthy cadence (v6.1 2026-08-27, v5.3.6 2026-09-16). A camera firmware must
  be rebased every ~2.5 years to stay patched; hobby projects rarely do.
- **Hardware H.264 encoder is open**: `esp-h264-component` is Apache-2.0 and
  the P4 hardware path is C source **[fetched]**. This corrects the first
  pass, which only saw the older software OpenH264 port.
- **ISP algorithms are binary**: in `esp-video-components`, `esp_ipa` ships
  prebuilt archives per IDF major under the non-OSI "Espressif MIT" licence,
  with only a detect stub in `src/` **[fetched]**. AWB/AE/denoise are blobs
  tied to IDF versions. Same structural problem as rkaiq, smaller.
- **RTSP/ONVIF:** the esp-adf request for a P4 RTSP example (Apr 2025) notes
  the existing implementation "is not open-source"; no staff reply visible.
  Espressif's `rtsp_demo` in esp-webrtc-solution pulls `esp_media_protocols`
  and `esp_peer` (ships a `libs` folder) from the registry, WiFi setup only.
  No Espressif ONVIF example.
- **Community since Feb 2026:** r4d10n/esp32p4-uvc-video: 19 commits, all
  between 2026-02-10 and 02-16, **nothing since** (dormant 7 months).
  ESP32CAM-ONVIF: 137 commits, active to 2026-08-28, Beta, **WiFi-only
  documented, no Ethernet, no IR-cut, and "H.264 RTP NAL packetization" still
  in progress** **[fetched]**. One WebRTC demo with 0 stars. That is the whole
  ecosystem.
- **PoE boards exist** *[snippets]*: Waveshare ESP32-P4-ETH with PoE module
  (~$24, camera kit ~$33), Olimex POEv3 add-on, Olimex ESP32-P4-PC (Ethernet
  with PoE, MIPI CSI, CERN-OHL-S, repo updated 2026-09-09 **[fetched]**).
  PoE class not verified.
- **Verdict:** a flash-and-go open P4 camera with PoE, H.264 RTSP/ONVIF and
  IR-cut control is plausible but on no funded roadmap. Estimate ~40% chance
  of a maintained turnkey firmware by 2029. **Re-evaluate in 2028.**

### Rockchip RV1106 / Luckfox, 10-year view

- **Mainline:** July 2026 series (Simon Glass) brings clocks, UART, SD/eMMC,
  SPI flash, GPIO, pinctrl and boots to a rootfs on Pico Mini B; August 2026
  series adds Ethernet, USB, I2C, DMA, PWM, "under review" *[snippets]*. An
  independent repo runs 6.18 with "no drivers or SoC function integrated"
  **[fetched]**. **No ISP or encoder driver upstream**; mainline `rkisp1`
  targets RK3288/RK3399-class ISPs; the target of the Aug 2026 `rkisp2` series
  is unverified.
- **Luckfox SDK:** kernel **5.10.160** **[fetched]**; 5.10 LTS EOL
  **2026-12-31** **[fetched]**. Cadence: one substantive merge per quarter
  through Aug 2025, then a 6-month gap to Mar 2026. Company founded 2020,
  Shenzhen, sells via Waveshare/Amazon *[snippet]*.
- **OpenIPC on RV1106 is the same architecture as on HiSilicon**: package
  `rockchip-osdrv-rv11xx` installs `mpp_vcodec.ko`, `rockit.ko`, `librkaiq.so`,
  `librockchip_mpp.so`, `librockit.so` for 5.10.160 **[fetched]**. Vendor
  kernel plus ISP/codec blobs, just on 5.10 instead of 3.10.
- **No IR-cut module verified.** SC3336 modules have no IR-cut (variant A only
  has pads for a fill-light board). The IMX415-98 IR-CUT camera is listed for
  Pi and Omni3576; RV1106 compatibility unverified.
- **Prices** *[snippet]*: Pico Ultra $18 base, +$4 WiFi, +$8 PoE module.
- **Verdict on "wait for mainline":** boot-level mainline arrives ~2027, but
  ISP and encoder, the parts a camera needs, have no visible upstream work.
  A blob-free RV1106 camera before 2029 is unlikely. Plan on 5.10.160 + blobs
  for the life of the node. Still the cheapest wired node and the youngest
  vendor kernel among the reflash routes.

### USB UVC IR cameras, 10-year view

- `uvcvideo` is actively maintained in mainline (fixes for v7.0 in Mar 2026,
  UVC 1.5 quirks 2026) *[snippet]*. Nothing on the camera is ever patched or
  needs to be.
- ELP (Shenzhen Ailipu, ~10 years) still lists H.264 IR models at $51 to $69
  *[snippets]*; stock not verified. **Treat MJPEG as the durable format** and
  H.264-over-UVC as a bonus that can vanish (Logitech precedent).
- 100 m USB 2.0 over Cat5e/6 extenders exist (StarTech, INOGENI) but users
  report intermittent failures and cheap units that only pass low resolutions.
  Budget ~$40 per active powered extender and expect it to be the least
  reliable component.
- Host: bandwidth is not the issue (MJPEG 1080p30 ~40 to 80 Mbps); decode
  is. Pi 5 has no H.264 hardware decode, so six streams are CPU-bound (~1
  core per 1080p MJPEG stream, estimate). An N100 with QSV handles it.
- **Verdict:** the most future-proof software and the lowest maintenance of
  any route, but the least deployable. Right for 1 to 2 cameras within 5 m of
  the NVR; wrong for whole-house coverage.

### Failure-mode sources

IR LEDs: 20,000 to 30,000 h before output is too weak (5.7 to 11 years at
12 h/night); LED-driver electrolytics fail 3 to 4× sooner. Wyze v3: firmware
"destroying 32 GB SD cards", spontaneous recording stops, 180-day warranty.
Pi SD: corruption "after a few months" under continuous writes. PoE:
indoor injectors in outdoor enclosures "usually fail" (enclosure runs 2×
ambient), voltage sag under load, surge on metal poles. HiSilicon: 60% (2018)
→ <4% (2021). All *[snippets]*.

### Not verified (section 6)

- Pi OS Trixie release date and the "critical kernel updates only" policy;
  Camera Module 3 to Jan 2030; any Pi statement on restoring hardware encode;
  IMX708 lifecycle; measured Pi 5 CPU for 1080p25 software encode.
- Espressif's P4 end date; whether `esp_media_protocols` is binary (registry
  blocked; relies on first pass); PoE class of the P4 boards.
- RV1106 mainline series review state; `rkisp2` target SoC; IMX415-98 IR-CUT
  compatibility with Pico boards; Luckfox founding year and prices.
- ELP company age, current H.264 IR models and prices; Logitech C920 story.
- Axis LTS windows, prices, wattage, warranty.
- All lifespans, maintenance hours and per-node wattages except the three
  supplied in the brief.

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

## Not verified this pass (sections 1, 2, 3, 7, 8; sections 4 to 6 carry their own lists)

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
