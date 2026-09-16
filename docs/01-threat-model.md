# 01 — Threat model: what "no cloud, no spyware" has to mean

Before picking hardware, define the enemy. "No cloud" is not one property; it is
five separate behaviours that a camera or NVR must be *unable* to perform, not
merely "configured not to". Configuration can be reset by a firmware update.
Inability cannot.

## The five behaviours to eliminate

| # | Behaviour | Where it lives | Real-world example |
|---|-----------|----------------|--------------------|
| 1 | Outbound telemetry / "phone home" | Camera firmware, NVR software | Vendor apps reporting uptime, model, IP, thumbnails |
| 2 | Cloud relay / P2P hole punching (UPnP, STUN, vendor relay) | Camera firmware | "Scan the QR code to view from anywhere" cameras. They punch through your NAT by design. |
| 3 | Auto-join of foreign networks | Any device with a WiFi radio and vendor firmware | Smart TVs / IoT joining open or previously-seen SSIDs without asking |
| 4 | Silent firmware auto-update | Camera and NVR | The update can re-enable 1 to 3 after you disabled them |
| 5 | Covert side channels | Camera firmware | DNS lookups, NTP to vendor servers, "check connectivity" pings, all usable to exfiltrate |

A device that runs the manufacturer's firmware can do all five, and you cannot
audit it. That is the whole argument for open firmware: not that open code is
bug-free, but that the code path for "send to vendor" does not exist to be
re-enabled.

## Trust boundaries in the target design

```
 [cameras]  --PoE/Ethernet-->  [camera VLAN, no gateway]  <--only NVR may enter--  [NVR box]
                                                                                        |
                                                                                 [LAN / you]
                                                                                        |
                                                                              [WireGuard you own]
                                                                                        |
                                                                                 [your phone]
```

Rules that fall out of the diagram:

1. **Cameras get no route to the internet.** No default gateway handed out by
   DHCP, and a firewall rule that drops everything from the camera VLAN that is
   not addressed to the NVR. Both, not one. The DHCP trick alone is defeated by
   a static route in firmware.
2. **Cameras are wired.** A camera with a WiFi radio can join a network you do
   not control. A camera with only an RJ45 jack cannot. If a camera must be
   WiFi (ESP32 class), give it a dedicated SSID on the camera VLAN and accept the
   radio as residual risk.
3. **The NVR is the only thing that talks to cameras**, and the NVR itself has
   no cloud account. Its updates are pulled by you, on purpose, from a source you
   chose.
4. **Remote access terminates on a VPN you run** (WireGuard on your router or a
   VPS you rent). Anything with a vendor "coordination server" or "tunnel"
   (Tailscale SaaS, Cloudflare Tunnel, vendor P2P) reintroduces a third party
   into the path, even if the video itself is encrypted.
5. **Time comes from you.** Run an NTP server on the NVR or router and point
   cameras at it. Otherwise a camera's "harmless" NTP query is the one packet
   you allowed out, and an exfil channel.

## What this design does NOT protect against

Be honest about the residual risk so nobody oversells it:

- **Physical access** to a camera or the NVR. Open firmware does not encrypt
  footage at rest by default.
- **Bugs in the open firmware or NVR.** They exist. The difference is that the
  fix does not depend on a vendor's roadmap, and there is no *intentional*
  exfil path.
- **Closed ISP / codec blobs inside "open" camera firmware.** OpenIPC and
  Thingino run an open Linux userland, but the image signal processor and often
  the kernel drivers are vendor binaries. They cannot phone home on their own
  (they have no network access; the open userland owns the network stack), but
  they are not auditable. See `03-open-camera-firmware.md`.
- **The NVR box's own OS.** Use a plain Debian/Ubuntu/Proxmox install you
  control, not a vendor appliance image.

## Definition of done for this project

The system is "no cloud" when all of these are true and have been *tested*, not
assumed:

- [ ] A packet capture on the camera VLAN uplink for 24 h shows zero packets to
      any destination other than the NVR and your NTP server.
- [ ] Every camera runs firmware whose source you can read (OpenIPC, Thingino,
      ESPHome/esp32-camera, Raspberry Pi OS + rpicam, etc.).
- [ ] The NVR has no account with any vendor, and its outbound firewall log is
      empty except for updates you triggered.
- [ ] Remote viewing works only through your own WireGuard tunnel.
- [ ] Pulling the internet uplink changes nothing about recording or local
      viewing.
