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

<!-- SECTIONS 5 6 9 PENDING -->

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
