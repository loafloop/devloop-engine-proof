# 06 — Network isolation and cloud-free remote access

Research date: 2026-09-16. Firewall syntax verified against OpenWrt's default
`firewall.config` and OPNsense's official how-tos; the camera-specific rules
are ours. UniFi steps are from general knowledge (help.ui.com was blocked).

## The pattern

1. **Dedicated camera VLAN on a PoE switch.** Static IPs or DHCP reservations.
   The VLAN gets **no route to WAN**: no default gateway *and* a firewall zone
   with no forwarding rule to WAN. Both. The DHCP trick alone is defeated by a
   static route baked into firmware. Frigate's docs confirm the NVR side needs
   nothing external: "Frigate is designed to run locally and does not require a
   persistent internet connection for core functionality"
   ([network_requirements.md](https://github.com/blakeblackshear/frigate/blob/dev/docs/docs/frigate/network_requirements.md)).
2. **NVR access:** either dual-home the NVR (second NIC or VLAN sub-interface in
   the camera VLAN, the classic Blue Iris dual-NIC pattern) or allow **only**
   NVR-IP → camera VLAN on TCP 554 (RTSP) and 80/8000 (ONVIF/HTTP) with stateful
   return traffic. Nothing camera-initiated leaves the VLAN.
3. **Kill DNS and NTP exfiltration.** Reflashed commercial cameras often
   hard-code public DNS/NTP. Drop all port 53 and 123 to WAN. Run local NTP on
   the router: OPNsense ntpd listens on all interfaces, constrain with rules
   ([ntpd.rst](https://github.com/opnsense/docs/blob/master/source/manual/ntpd.rst));
   or chrony with `allow 192.168.50.0/24` and `local stratum 10` so it keeps
   serving when the router itself is offline. Point cameras at the router IP,
   or DNAT UDP/123 from the camera VLAN to it.
4. **Discovery is link-local.** ONVIF WS-Discovery is multicast
   239.255.255.250:3702; mDNS is 224.0.0.251:5353. Neither crosses a VLAN, so
   auto-discovery from an NVR on another VLAN fails. Use manual RTSP URLs and
   static IPs. **Do not** add an mDNS/SSDP reflector between camera VLAN and LAN.
5. **UPnP off** on the router and on any camera firmware that has it.
6. **WiFi cameras** (Wyze on Thingino, ESP32, Pi Zero 2 W): a separate SSID
   mapped to the camera VLAN with client isolation. Treat identically.

## Firewall examples

### OpenWrt (UCI)

```
config zone
    option name 'cams'
    list network 'cams'          # the camera VLAN interface
    option input 'REJECT'        # cameras may not talk to the router by default
    option output 'ACCEPT'
    option forward 'REJECT'
    option log '1'               # log drops -> logread

# Deliberately NO "config forwarding" from cams to wan  => no internet

config rule
    option name 'NVR-to-cams'
    option src 'lan'
    option src_ip '192.168.1.10'   # the NVR
    option dest 'cams'
    option dest_port '554 80 8000'
    option proto 'tcp'
    option target 'ACCEPT'

config rule
    option name 'cams-DHCP-NTP-to-router'
    option src 'cams'
    option proto 'udp'
    option dest_port '67 68 123'
    option target 'ACCEPT'
```

### OPNsense

Pattern from the official guest-network how-to
([guestnet.rst](https://github.com/opnsense/docs/blob/master/source/manual/how-tos/guestnet.rst)).
On interface CAMS, in order:

1. Pass UDP, CAMS net → CAMS address, ports 67 and 123 (DHCP, NTP).
2. Block (log) CAMS net → LAN net (repeat per local net or use an alias).
3. Block (log) CAMS net → This Firewall.
4. **No pass-to-any rule**, so the implicit default deny stops internet.

On LAN: Pass TCP from NVR host → CAMS net, ports 554, 80, 8000.
Verify in Firewall → Log Files → Live View. Logging must be enabled per rule;
only the first packet of a state is logged
([logging_firewall.rst](https://github.com/opnsense/docs/blob/master/source/manual/logging_firewall.rst)).

### UniFi (unverified, general knowledge)

Create a "Cameras" network/VLAN. In Network 9.x zone-based firewall, put it in
its own zone; set Cameras → External and Cameras → Internal to Block; add one
Allow policy Internal → Cameras with source = NVR IP. Disable UPnP under
Internet settings. Camera switch ports: native VLAN Cameras only.

## Remote viewing without cloud

| Option | Third party in the path? | Notes |
|---|---|---|
| **WireGuard on your own router** | **No** | OPNsense road-warrior how-to verified: instance on UDP 51820, tunnel 10.10.10.1/24, peers with /32 Allowed IPs, WAN rule for 51820, MSS clamp 1380/1360 ([wireguard-client.rst](https://github.com/opnsense/docs/blob/master/source/manual/how-tos/wireguard-client.rst)). OpenWrt equivalent exists. Client reaches NVR/Frigate UI over the tunnel; camera VLAN stays unreachable. Behind CG-NAT: a cheap VPS as WireGuard hub sees only encrypted packets. **This is the recommendation.** |
| Home Assistant WireGuard add-on | No | Runs a WireGuard **server** on the HA host, generates peer configs and QR codes ([hassio-addons/addon-wireguard](https://github.com/hassio-addons/addon-wireguard)). |
| **Headscale** | No | Self-hosted implementation of the Tailscale control server, BSD-3, ~44k stars, uses stock Tailscale clients ([juanfont/headscale](https://github.com/juanfont/headscale)). More moving parts than plain WireGuard, but gives you the Tailscale client UX. |
| NetBird self-hosted | No | WireGuard overlay with self-hosted management/signal/relay, BSD-3 + AGPL. Needs a Linux VM, public domain, TCP 80/443 + UDP 3478, Docker Compose, identity provider ([netbirdio/netbird](https://github.com/netbirdio/netbird)). |
| Tailscale hosted | **Yes, control plane** | Data plane is peer-to-peer WireGuard; DERP relays forward only encrypted packets ([derp README](https://github.com/tailscale/tailscale/blob/main/derp/README.md)). But device keys, names, IPs, ACLs and your identity-provider login live in Tailscale's closed SaaS. Fine for many people. Not "no cloud". |
| Cloudflare Tunnel | **Yes, sees plaintext** | `cloudflared` makes outbound connections to Cloudflare's network; in Full/Full-strict modes there are two TLS legs, so Cloudflare terminates TLS and sees the video ([tunnel docs](https://github.com/cloudflare/cloudflare-docs/blob/production/src/content/docs/cloudflare-one/networks/connectors/cloudflare-tunnel/index.mdx)). Their terms also restrict video through the proxy (unverified this round). **Do not use.** |
| Vendor P2P / QR-code apps | Yes, by design | The thing this whole project exists to avoid. |

## Proving a camera cannot reach the internet

The definition of done in `01-threat-model.md` requires a test, not a belief.

1. **Capture on the camera VLAN uplink** (router or NVR):

   ```sh
   tcpdump -ni <cams-if> 'src net 192.168.50.0/24 and not dst net 192.168.50.0/24 and not dst host <router-ip>'
   ```

   Should stay silent apart from the NTP/DHCP you allowed. Anything to
   53/123/443/8883 or a vendor IP is a leak attempt. Leave it running 24 h;
   some firmware only tries at boot or on a schedule.
2. **Firewall logs:** OPNsense Live View filtered on the CAMS interface;
   OpenWrt `logread -f | grep REJECT` with `option log '1'`.
3. **From the camera's own shell** (open firmware gives you one):
   `ping -c1 1.1.1.1`, `nslookup example.com`, `curl -m5 https://example.com`
   must all fail. Confirm resolver and NTP config point at the router.
4. **Periodically:** `conntrack -L | grep 192.168.50.` on the router, and check
   switch MAC tables so no camera has migrated to another VLAN.
5. **Pull the WAN cable.** Recording and local viewing must not change.
