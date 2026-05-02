---
title: "Ubiquiti 2Gb Home Setup for Empire Internet Access"
date: 2026-05-01
updated: 2026-05-01 21:25 ET
summary: "Minimal Ubiquiti build for Empire Access fiber (up to 2 Gig symmetric in upstate NY): UniFi Cloud Gateway Fiber as the router (10G SFP+ WAN, integrated PoE), one U7 Pro AP, one short Cat6a patch run, no extra switch needed. Total around $498 plus cables (or $563 with the optional SFP+ to RJ45 adapter). store.ui.com is the cheapest source for both gateway and AP; Newegg/B&H/Adorama match the price but rarely undercut Ubiquiti."
---

## TL;DR

Recommended minimal build for 2 Gbps service over Empire Access fiber, future-proofed for 10G:

- 1x **UniFi Cloud Gateway Fiber (UCG-Fiber)** at $279 ([source](https://store.ui.com/us/en/products/ucg-fiber)) — 1x 10G SFP+ WAN + 1x 10GbE RJ45 WAN, four 2.5GbE LAN + one 10GbE RJ45 LAN + two 10G SFP+ LAN, **30W PoE budget on board**, 5 Gbps IDS/IPS ([source](https://techspecs.ui.com/unifi/cloud-gateways/ucg-fiber)).
- 1x **UniFi U7 Pro** Wi-Fi 7 access point at $189 ([source](https://store.ui.com/us/en/category/all-wifi)) — 2.5GbE PoE+ uplink, ceiling/wall mount, 6 spatial streams ([source](https://techspecs.ui.com/unifi/wifi/u7-pro)). Powered directly off the UCG-Fiber's PoE output — no separate adapter needed.
- 1x **SFP+ to RJ45 Adapter (UACC-CM-RJ45-MG)** at $65 ([source](https://store.ui.com/us/en/products/uacc-cm-rj45-mg)) — supports 1G/2.5G/5G/10G over Cat6a; lets you plug the RJ45 handoff from Empire's ONT into the UCG-Fiber's 10G SFP+ WAN port. Skip this only if Empire offers a direct fiber/SFP handoff at install time, OR if you use the UCG-Fiber's 10GbE RJ45 WAN port directly (recommended unless you want to keep RJ45 free for a 10G client).
- ~$30 in **Cat6a** patch cables for the runs longer than a meter; Cat6 is fine inside a single rack.
- **Total: ~$563 hardware** plus cables (or ~$498 if you skip the SFP+ adapter and use the 10GbE RJ45 WAN port directly). Empire Access provides the ONT for free; you replace their router with the UCG-Fiber.

**Why UCG-Fiber over UCG-Max:** UCG-Fiber adds a 10G SFP+ WAN port (so you can terminate fiber directly if Empire ever offers an SFP handoff, or use a multigig RJ45 SFP+ adapter today), bumps IDS/IPS from 2.3 Gbps to 5 Gbps (zero throughput loss even if Empire doubles to 4 Gig), and includes a 30W PoE budget so the U7 Pro AP runs straight off the gateway with no separate PoE injector. The +$80 over UCG-Max also gets a 10GbE RJ45 LAN port for a future NAS or 10G workstation. Worth it for a Pete-style "buy once, hold for 5+ years" build.

Important note on Empire's current speed tiers: Empire Access tops out at **2 Gig symmetric** for residential right now ([source](https://www.empireaccess.com/faq/internet-speeds/)). They do not offer 5 Gig residential as of May 2026. UCG-Fiber's 10G WAN is overspecced for today's 2 Gig service — that's the point: it gives you a 5x runway on WAN speed plus integrated PoE and stronger IDS/IPS, all for $80 over the bare-minimum UCG-Max pick.

## Current Status

- **ISP fit:** Empire Access "Elite" plan is 2 Gbps symmetric. UCG-Fiber's 10G SFP+ / 10GbE RJ45 WAN handles it with massive headroom for future upgrades ([source](https://www.empireaccess.com/faq/internet-speeds/)).
- **Gateway pick:** UCG-Fiber ($279), 10G SFP+ + 10GbE RJ45 WAN, 4x 2.5GbE LAN + 10GbE RJ45 LAN + 2x 10G SFP+ LAN, **30W PoE budget**, 5 Gbps IDS/IPS ([source](https://techspecs.ui.com/unifi/cloud-gateways/ucg-fiber)).
- **AP pick:** U7 Pro ($189) ceiling AP, Wi-Fi 7, 2.5GbE uplink, PoE+ powered directly off the UCG-Fiber's PoE output ([source](https://techspecs.ui.com/unifi/wifi/u7-pro)).
- **Port count:** UCG-Fiber gives you 7 LAN ports total (4x 2.5GbE + 1x 10GbE RJ45 + 2x 10G SFP+). One port for the AP leaves six for wired clients — comfortably covers 4-6 devices with no switch needed.
- **Empire ONT handoff:** Use the UCG-Fiber's **10GbE RJ45 WAN port** directly with a Cat6a patch cable from Empire's ONT — simplest path, no transceiver required. Optional: pick up the UACC-CM-RJ45-MG SFP+ adapter ($65, supports 1G/2.5G/5G/10G over Cat6a) ([source](https://store.ui.com/us/en/products/uacc-cm-rj45-mg)) if you'd rather use the SFP+ port for WAN and keep the 10G RJ45 free for a future LAN client.
- **Two SSIDs:** UniFi Network app supports unlimited SSIDs each tagged to a VLAN; Main + IoT (or Main + Guest) is a 10-minute config ([source](https://help.ui.com/hc/en-us/articles/26136823938583-Creating-UniFi-WiFi-SSIDs)).
- **CLI/SSH:** UniFi gateways DO have SSH (`root@<gateway-ip>`, password set in Network app), but it is a debug tool, not a primary config interface. UniFi is GUI-first; SSH is for diagnostics, advanced packet captures, and `unifi-os` shell access ([source](https://help.ui.com/hc/en-us/articles/204909374-Connecting-to-UniFi-with-Debug-Tools-SSH)).
- **Total cost:** ~$498 hardware before cables (without SFP+ adapter), or ~$563 with the SFP+ adapter.
- **Lead times:** UCG-Fiber and U7 Pro both available at store.ui.com as of 2026-05-01 ([source](https://store.ui.com/us/en/category/all-cloud-gateways)) ([source](https://store.ui.com/us/en/category/all-wifi)).

## Findings

### 1. Empire Internet Access — What You're Connecting To

Empire Access (residential brand: "Empire Fiber Internet") is a regional fiber ISP serving 17 New York counties including Tompkins (Ithaca), Steuben, Chemung, Cortland, Ontario, Monroe, Wayne, Yates, and others ([source](https://www.empireaccess.com/about-us/communities-we-serve/)).

**Speed tiers (residential, symmetric, as of March 2024 upgrade):** ([source](https://www.empireaccess.com/empire-access-upgrades-residential-internet-upload-speeds-by-1000-percent/)) ([source](https://www.empireaccess.com/faq/internet-speeds/))

| Plan | Speed |
|------|-------|
| Essentials | 500/500 Mbps |
| Enhanced | 1/1 Gbps |
| Elite | 2/2 Gbps |

**No 5 Gig or 10 Gig residential tier exists** as of May 2026. The original task brief mentioned "2-5 Gbps" — that's not Empire's current offering. Plan around the 2 Gig Elite tier as the ceiling.

**Equipment they install:** Empire installs an ONT (optical network terminal) on the wall. The ONT converts the fiber light signal to Ethernet, then hands off to a Wi-Fi router that they also provide ([source](https://www.empireaccess.com/understanding-fiber-internet-and-how-its-installed/)). Empire bundles the equipment "free of charge" but does not publicly document a Bring-Your-Own-Router policy ([source](https://www.empireaccess.com/faq/what-equipment-do-i-need-for-empire-fiber-internets-internet-service/)).

**Practical reality (medium confidence):** Most ONTs hand off via standard Ethernet (RJ45). Once Empire's ONT is on your wall with an Ethernet port, you can plug ANY router into it, including a UCG-Max. Empire's router becomes optional — leave it on a shelf, or politely decline at install. Other Calix-based fiber ISPs work the same way ([source](https://support.avative.com/hc/en-us/articles/40119741588116-WiFi-Routers-Mesh-Extenders)). Confirm with Empire's installer on day one.

**Open question for Pete:** Ask the Empire installer (1) what model ONT they're installing (likely a Calix unit or a Calix Gigaspire combo), (2) whether the ONT has a separate Ethernet jack you can use directly, and (3) whether their managed Wi-Fi service (CommandIQ, ProtectIQ) is bundled or optional. If the "router" they install is actually a Calix Gigaspire (ONT + router combo), ask if it can be put in pure-bridge / IP-passthrough mode so your UCG-Max gets the public IP.

### 2. Gateway Selection (2.5GbE+ WAN)

Current Ubiquiti cloud gateway lineup with 2.5GbE WAN or better, available as of 2026-05-01 ([source](https://store.ui.com/us/en/category/all-cloud-gateways)):

| Model | SKU | Price | WAN | LAN | PoE Budget | IDS/IPS | Notes |
|-------|-----|-------|-----|-----|------------|---------|-------|
| Cloud Gateway Max | UCG-Max | $199 | 1x 2.5GbE | 4x 2.5GbE + 1x 1GbE | None | 2.3 Gbps | Cheapest 2.5G WAN option; needs PoE adapter for AP ([source](https://techspecs.ui.com/unifi/cloud-gateways/ucg-max)) |
| Cloud Gateway Fiber | UCG-Fiber | $279 | 1x 10G SFP+ + 1x 10GbE RJ45 | 4x 2.5GbE + 1x 10GbE RJ45 + 2x 10G SFP+ | **30W** | 5 Gbps | **Recommended** — 10G ready, integrated PoE, 5 Gbps IDS/IPS ([source](https://techspecs.ui.com/unifi/cloud-gateways/ucg-fiber)) |
| Express 7 | UX7 | $199 | 1x 10GbE | 1x 2.5GbE | None | 2.3 Gbps | Has built-in WiFi 7, only 1 LAN port — not enough for 4-6 wired devices ([source](https://techspecs.ui.com/unifi/cloud-gateways/ux7)) |
| Dream Router 7 | UDR7 | $279 | (unspecified by store, has 10G) | (unspecified) | (PoE switch built in) | 2.3 Gbps | All-in-one router + WiFi 7 + microSD storage ([source](https://store.ui.com/us/en/products/udr7)) |
| Dream Machine Pro Max | UDM-Pro-Max | $599 | 10G | 10G | None | 5 Gbps | Rack-mount, way too much for a home with 2 Gig service ([source](https://store.ui.com/us/en/category/all-cloud-gateways)) |

**Pick: UCG-Fiber** ($279). Reasons:

1. **10G SFP+ WAN port** terminates fiber directly if Empire ever offers an SFP handoff, or accepts a multigig RJ45 SFP+ adapter today (UACC-CM-RJ45-MG, 1G/2.5G/5G/10G, $65) ([source](https://store.ui.com/us/en/products/uacc-cm-rj45-mg))
2. **Plus a 10GbE RJ45 WAN** port — for the simple Empire-ONT install, just patch from the ONT's RJ45 jack into this port and skip the SFP+ adapter entirely
3. **Integrated 30W PoE budget** — the U7 Pro AP draws ~21W at PoE+, runs straight off the gateway with no separate injector ([source](https://techspecs.ui.com/unifi/cloud-gateways/ucg-fiber))
4. **5 Gbps IDS/IPS routing** (vs UCG-Max's 2.3 Gbps) — full speed even if Empire doubles to 4 Gig, with all CyberSecure signatures enabled
5. **Seven LAN ports** (4x 2.5GbE + 1x 10GbE RJ45 + 2x 10G SFP+) — covers 4-6 wired clients plus the AP with no extra switch, and includes 10G for a future NAS or workstation
6. Buy-once-hold-five-years math: +$80 over UCG-Max, but eliminates the $15 PoE adapter and the future need for a 10G upgrade

**Skip:** UCG-Max ($199) — works for today but caps at 2.5G WAN, has no PoE output (forcing the +$15 adapter), and tops out at 2.3 Gbps with IDS/IPS on. The $65 delta to UCG-Fiber buys a lot of headroom. UCG-Ultra ($129) has only 1GbE WAN and would bottleneck Empire's 2 Gig service to half speed ([source](https://store.ui.com/us/en/category/all-cloud-gateways)). UDM-Pro/SE/Beast/Industrial are rack-mount enterprise units, not minimal home setups.

### 3. Access Point Selection

UniFi's Wi-Fi 7 lineup, in stock as of 2026-05-01 ([source](https://store.ui.com/us/en/category/all-wifi)):

| Model | SKU | Price | Uplink | Streams | Mount | Best for |
|-------|-----|-------|--------|---------|-------|----------|
| U7 Lite | U7-Lite | $99 | 2.5GbE | 4 (dual-band, no 6 GHz) | Ceiling/wall | Budget single-AP coverage ([source](https://techspecs.ui.com/unifi/wifi/u7-lite)) |
| U7 Pro | U7-Pro | $189 | 2.5GbE | 6 (tri-band incl. 6 GHz) | Ceiling | **Recommended** ([source](https://techspecs.ui.com/unifi/wifi/u7-pro)) |
| U7 Pro XG | U7-Pro-XG | $199 | 10GbE | 6 (tri-band) | Ceiling | Only worth it if your AP uplink can do 10G ([source](https://techspecs.ui.com/unifi/wifi/u7-pro-xg)) |
| U7 Pro Wall | U7-Pro-Wall | $199 | (unspecified) | 6 | Wall | Wall mount form factor ([source](https://store.ui.com/us/en/category/all-wifi)) |
| U7 In-Wall | U7-IW | $149 | 2.5GbE PoE+ in | 4 (dual-band) | US wall plate | Replaces an Ethernet wall jack with an AP + 3-port switch ([source](https://techspecs.ui.com/unifi/wifi/u7-iw)) |

**Pick: U7 Pro** ($189). Reasons:

1. Wi-Fi 7 with 6 GHz band for clean spectrum — current high-end client devices (modern phones, laptops) will use this
2. 2.5GbE uplink matches the gateway's LAN ports — full speed AP traffic
3. PoE+ powered (~21W) — one Ethernet cable carries data and power ([source](https://techspecs.ui.com/unifi/wifi/u7-pro))
4. 6 spatial streams handles a busy house (modern Mac, iPhones, smart TVs, IoT sensors) without breaking a sweat
5. Most homes only need ONE good AP. A 2-story 2,000 sq ft home is well within range of a single ceiling-mounted U7 Pro

**Powering the AP:** With UCG-Fiber as the gateway, the AP runs straight off the gateway's PoE output — no separate adapter needed. The U7 Pro draws ~21W at PoE+, well under the UCG-Fiber's 30W PoE budget ([source](https://techspecs.ui.com/unifi/cloud-gateways/ucg-fiber)). Just plug a single Cat6 run from any of the UCG-Fiber's 2.5GbE LAN ports to the U7 Pro. (If you ever swap in UCG-Max, you'd need the $15 U7 Pro PoE+ Adapter or a small PoE switch — UCG-Max has no PoE output.)

**Skip the U7 Lite** unless budget is tight: it has no 6 GHz band, which is the main reason to pick Wi-Fi 7 in 2026.

**Skip U7-IW** unless Pete already has Ethernet drops at wall plates and wants a flush-mount AP.

### 4. Switch — Needed or Not?

UCG-Fiber has 7 LAN ports total: 4x 2.5GbE RJ45 + 1x 10GbE RJ45 + 2x 10G SFP+ ([source](https://techspecs.ui.com/unifi/cloud-gateways/ucg-fiber)). Pete wants 4-6 wired devices.

**Math:**
- 1 port → AP (the U7 Pro takes a 2.5GbE LAN port; PoE travels in the same cable from the gateway)
- Up to 6 ports remaining → wired clients (Mac, NAS at 10G if you have one, gaming console, etc.)
- Comfortably supports **6 wired clients + 1 AP** with no extra switch

**No switch needed for the core build.** UCG-Fiber's port count covers the use case directly.

If Pete later expands to **8+ wired clients** (or wants more PoE ports for additional APs/cameras), options:

| Model | SKU | Price | Ports | PoE | Best for |
|-------|-----|-------|-------|-----|----------|
| UniFi Flex Mini 2.5G | USW-Flex-2.5G-5 | $49 | 5x 2.5GbE | None | Small fanless desktop switch, all 2.5G ([source](https://store.ui.com/us/en/category/all-switching)) |
| UniFi Flex 2.5G PoE | USW-Flex-2.5G-8-PoE | $199 | 8x 2.5GbE + 1x 10GbE/SFP+ uplink | 4x PoE++ ports | More PoE for additional APs or cameras ([source](https://store.ui.com/us/en/category/all-switching)) |
| UniFi Lite 8 PoE | USW-Lite-8-PoE | $109 | 8x 1GbE | 4x PoE+ ports | Cheaper but 1G only (bottlenecks 2.5G clients) — skip ([source](https://store.ui.com/us/en/category/all-switching)) |

A 10G SFP+ uplink switch (e.g., USW-Flex-2.5G-8-PoE) plugs into one of UCG-Fiber's 10G SFP+ LAN ports for a non-blocking expansion path.

### 5. Two WiFi Networks (VLAN + SSID Setup)

UniFi Network app fully supports multiple SSIDs each tagged to a separate VLAN with traffic isolation ([source](https://help.ui.com/hc/en-us/articles/26136823938583-Creating-UniFi-WiFi-SSIDs)) ([source](https://help.ui.com/hc/en-us/articles/9761080275607-Creating-Virtual-Networks-VLANs)). Workflow:

1. **Settings → Networks → Create New Network** → make a virtual network for "IoT" with its own VLAN ID (e.g., 20) and DHCP scope. Repeat for "Guest" if you want a third network.
2. **Settings → Wi-Fi → Add New Wi-Fi Network** → create SSID "HomeWiFi" mapped to the default LAN; create SSID "IoT" mapped to the IoT VLAN. Each SSID can have its own password, band steering rules, and access controls.
3. **For Guest specifically:** UniFi auto-creates the firewall rules to isolate guest traffic from your LAN — toggle "Guest network" on in the network settings ([source](https://itman.ae/2025/07/07/set-up-secure-guest-iot-network-on-unifi-aps/)).
4. **For IoT:** Toggle "Client device isolation" on the IoT SSID so smart bulbs and cameras can't see each other ([source](https://help.ui.com/hc/en-us/articles/26136823938583-Creating-UniFi-WiFi-SSIDs)). Add a firewall rule under Settings → Firewall & Security → Rules → LAN IN to block IoT VLAN from initiating to LAN VLAN.
5. **Both SSIDs broadcast from the single U7 Pro** — one AP can serve unlimited SSIDs/VLANs.

Time to configure: 10-15 minutes once the gateway is adopted.

### 6. CLI/SSH Support — The Honest Answer

This is the section to read carefully. Pete likes terminal-driven config; UniFi is **not** that ecosystem. Here's the truth:

**What you DO get with SSH:**
- SSH into the gateway as `root` with the password set in the UniFi Network app under Settings → System → Device Authentication ([source](https://help.ui.com/hc/en-us/articles/204909374-Connecting-to-UniFi-with-Debug-Tools-SSH))
- A standard Linux shell (the gateway runs UniFi OS, a Debian-based Linux)
- Read access to logs, network state, packet captures (`tcpdump`), routing tables
- The `unifi-os shell` command on Cloud Keys/Dream Machines drops you into the underlying Debian environment for power-user work
- Browser-based "Debug Terminal" under Settings → System (the Ubiquiti-recommended path) ([source](https://help.ui.com/hc/en-us/articles/204909374-Connecting-to-UniFi-with-Debug-Tools-SSH))

**What you DON'T get:**
- A real network CLI like Cisco IOS or Juniper Junos. There is no `configure terminal` mode for managing VLANs, SSIDs, firewall rules from CLI in any supported way.
- Ubiquiti explicitly says SSH is for diagnostics: "SSH is not recommended unless instructed by Support Engineers as part of advanced troubleshooting" ([source](https://help.ui.com/hc/en-us/articles/204909374-Connecting-to-UniFi-with-Debug-Tools-SSH)).
- Configuration changes made via SSH may be overwritten on the next controller sync. The controller is the source of truth.

**Third-party tools that bring some terminal/automation back:**
- **Python `pyunifi` / `unifi-controller-api`** packages — wrap the Network app's REST API for scripting. You can pull device lists, force device adoption, push some config from Python.
- **`unpoller`** — Prometheus exporter that scrapes UniFi metrics into Grafana. Good for dashboards.
- **Home Assistant integration** — full control of the UniFi controller via HA automations.
- **Direct REST API** — UniFi has an undocumented but well-reverse-engineered REST API at `https://<controller>/api/`. You can curl it.

**Honest summary:** If Pete wants `terraform apply` for his network, UniFi is not it — try OPNsense or VyOS instead. If Pete wants a clean GUI with SSH for diagnostics and a REST API for automation when needed, UniFi is excellent. The fact that he runs Claude Code is more relevant for automation around UniFi (write a Python script that pulls daily client stats) than for in-the-box CLI config.

### 7. Cabling Notes

For 2.5GbE ([source](https://www.truecable.com/blogs/cable-academy/cat6-vs-cat6a)):
- **Cat5e:** Officially rated for 1G to 100m; 2.5G works on most installs up to ~50m but is not rated. Don't run new Cat5e in 2026.
- **Cat6:** Handles 2.5G easily up to 100m. Handles 10G up to ~55m.
- **Cat6a:** Handles 10G up to 100m with better shielding. The future-proof default.

**Recommendation:** Cat6 is fine for any 2.5G run in a typical home. Use Cat6a if (a) the run is over 50 ft and you want headroom for 10G later, or (b) the run goes through a noisy electrical environment. Pre-made patch cables in 1-3 ft, 7 ft, and 25 ft sizes from Monoprice or BlueJeans. Avoid cheap unbranded Amazon cables — copper-clad aluminum (CCA) is common and unreliable at 2.5G+.

For the run from the ONT/Empire equipment to the UCG-Max: short Cat6 patch is fine. For the run from the UCG-Max to the U7 Pro AP: Cat6 if under 50 ft, Cat6a if longer.

### 8. Setup Steps (Minimal)

1. **Schedule Empire install.** When the tech arrives, ask: (a) what's the ONT model, (b) is it standalone with an Ethernet handoff, (c) can I use my own router. If they install a Calix Gigaspire combo, ask for it in bridge/passthrough mode.
2. **Unbox UCG-Fiber.** Power it via the included 54V adapter. Plug the **10GbE RJ45 WAN port** into the ONT's Ethernet handoff with a Cat6a patch cable (simplest path). Or, if using the SFP+ adapter, install UACC-CM-RJ45-MG into a 10G SFP+ port and patch from the ONT into that.
3. **Connect a laptop** to one of the UCG-Fiber's 2.5GbE LAN ports OR connect via Wi-Fi to the temporary "UniFi" SSID it broadcasts during setup.
4. **Open the UniFi Network mobile app** (iOS or Android), scan the QR code on the back of the UCG-Fiber, follow the wizard. Create your UI.com SSO account if you don't have one.
5. **Plug in the U7 Pro.** Single Cat6 cable from any UCG-Fiber 2.5GbE LAN port directly to the U7 Pro. Power and data travel on the same cable thanks to the gateway's 30W PoE budget. The AP will appear in the UniFi app for adoption — tap Adopt.
6. **Configure 2 SSIDs** per Section 5. Run a speed test from a laptop on the main SSID; confirm you're hitting near 2 Gbps over 2.5GbE wired or close to gigabit on Wi-Fi.
7. **Plug in your 4-6 wired devices** to the remaining LAN ports (use the 10GbE RJ45 or 10G SFP+ ports if you have a NAS or workstation that supports 10G).
8. **(Optional) Enable SSH:** Settings → System → Device Authentication → set SSH password. Test from your Mac: `ssh root@<gateway-ip>`. Useful for diagnostics; don't try to manage config from here.

Total time, soup to nuts: 60-90 minutes including the install conversation with Empire.

## Recommended Bill of Materials

**Core build (up to 6 wired clients, AP powered off the gateway):**

| Qty | SKU | Item | Price | Link |
|-----|-----|------|-------|------|
| 1 | UCG-Fiber | UniFi Cloud Gateway Fiber | $279 | [store.ui.com](https://store.ui.com/us/en/products/ucg-fiber) |
| 1 | U7-Pro | UniFi U7 Pro Wi-Fi 7 AP | $189 | [store.ui.com](https://store.ui.com/us/en/products/u7-pro) |
| 1 | — | Cat6a patch cable kit (assorted) | ~$30 | Monoprice / BlueJeans |
| **Total** | | | **~$498** | |

Empire's RJ45 ONT handoff plugs directly into UCG-Fiber's 10GbE RJ45 WAN port. No transceiver required. AP runs off the UCG-Fiber's built-in 30W PoE budget — no PoE injector or PoE switch needed.

**Optional add-on if you want SFP+ WAN flexibility:**

| Qty | SKU | Item | Price | Link |
|-----|-----|------|-------|------|
| 1 | UACC-CM-RJ45-MG | SFP+ to RJ45 Adapter (1G/2.5G/5G/10G) | $65 | [store.ui.com](https://store.ui.com/us/en/products/uacc-cm-rj45-mg) |
| **New total** | | | **~$563** | |

Add this only if you'd rather wire Empire's RJ45 ONT into the 10G SFP+ port (frees the 10GbE RJ45 for a 10G LAN client), OR if Empire ever offers a direct fiber/SFP handoff and you want a hot-swappable transceiver path.

**If you later expand past 6 wired clients (add a downstream switch):**

| Qty | SKU | Item | Price | Link |
|-----|-----|------|-------|------|
| 1 | UCG-Fiber | UniFi Cloud Gateway Fiber | $279 | [store.ui.com](https://store.ui.com/us/en/products/ucg-fiber) |
| 1 | U7-Pro | UniFi U7 Pro Wi-Fi 7 AP | $189 | [store.ui.com](https://store.ui.com/us/en/products/u7-pro) |
| 1 | USW-Flex-2.5G-8-PoE | UniFi Flex 2.5G PoE 8-Port | $199 | [store.ui.com](https://store.ui.com/us/en/category/all-switching) |
| 1 | — | Cat6a patch cable kit | ~$30 | Monoprice / BlueJeans |
| **Total** | | | **~$697** | |

**Alternative all-in-one (gateway + AP + storage in one unit):**

| Qty | SKU | Item | Price |
|-----|-----|------|-------|
| 1 | UDR7 | UniFi Dream Router 7 | $279 |
| | | Cat6 cables | ~$30 |
| **Total** | | | **~$309** |

UDR7 is the cheapest path to a working setup with one device but you give up the dedicated 10G WAN options of UCG-Fiber, and the option to separate router from AP for better placement. Good for an apartment or small home; not as good for placing the AP centrally and the gateway by the ONT.

## Where to Buy / Best Deals

Quick honest summary: **Ubiquiti products almost never sell below MSRP at third-party retailers.** Authorized resellers (Amazon, B&H, Newegg, Adorama) match store.ui.com's price and occasionally do free shipping, but they don't undercut. There's no public "UI Store Refurbished" section at store.ui.com as of May 2026 (404 on the refurb URL). No "Early Access" pricing applies to UCG-Fiber or U7 Pro; both are mainline shipping products. Promo code aggregators (SimplyCodes, Tenere, etc.) showing "10% off Ubiquiti" or "70% off" coupons in May 2026 search results are bait. Ubiquiti does not run sitewide promos and these codes don't work at checkout.

**Bottom line: store.ui.com is the cheapest source for both gateway and AP at MSRP.** The only real lever is whether you'd rather pay Ubiquiti's shipping (calculated at checkout, often $10 to $15 for a small order) vs. Amazon Prime or Newegg free shipping for $0, which can flip the per-item math by a few dollars.

| Item | Best Source | Price | Notes |
|------|-------------|-------|-------|
| **UCG-Fiber** | [store.ui.com](https://store.ui.com/us/en/products/ucg-fiber) | $279 | MSRP. Newegg shows Adorama at $279 with free shipping ([source](https://www.newegg.com/p/0E6-003V-005X4)); Newegg's own stock was Sold Out at time of check. B&H carries it but no listed discount ([source](https://www.bhphotovideo.com/c/product/1884945-REG/ubiquiti_networks_ucg_fiber_cloud_gateway_fiber.html)). Amazon listings exist (B0DZSB9HYY, B0FD86XR36) but commonly show third-party sellers at markup. Tax applies in NY at all retailers. |
| **U7 Pro** | [Newegg](https://www.newegg.com/ubiquiti-networks-u7-pro/p/N82E16833664088) (free shipping) **or** [store.ui.com](https://store.ui.com/us/en/products/u7-pro) | $189 | Identical $189 at both. Newegg edges it out only if Ubiquiti's shipping fee on a single AP is more than Newegg's. If you're buying gateway + AP together at store.ui.com, the bundled shipping likely makes Ubiquiti the marginal winner. Amazon also stocks at $189 (B0CSR63FMH); B&H lists it at $189 as well ([source](https://www.bhphotovideo.com/c/product/1805878-REG/ubiquiti_networks_u7_pro_us_unifi_wifi_7_pro.html)). |
| **Cat6a patch cables (3-7 ft, 5-6 cables)** | Monoprice.com or Amazon | ~$25 to $40 total | Monoprice SlimRun Cat6A 30AWG is the standard recommendation: 3ft 5-pack (#15129/15130, ~$10) and 7ft 10-pack (#15162, ~$25). Cable Matters and TrueCABLE are both reputable alternatives on Amazon Prime. Avoid CCA (copper-clad aluminum). Only buy cables labeled "pure bare copper" or "100% copper conductor". Either Monoprice or Amazon Prime will land cheaper than buying cables from store.ui.com. |
| **UACC-CM-RJ45-MG** (optional) | [store.ui.com](https://store.ui.com/us/en/products/uacc-cm-rj45-mg) | $65 | Ubiquiti-only item; not stocked at meaningful discount elsewhere. Skip entirely if using the UCG-Fiber's 10GbE RJ45 WAN port directly (the recommended path). |

**Shipping notes:**
- store.ui.com: shipping calculated at checkout based on order weight/destination; no public free-shipping threshold. NY users pay sales tax.
- Amazon Prime: free 2-day shipping, NY sales tax applies.
- Newegg: free shipping on Newegg-shipped items, NY sales tax applies. Watch for third-party Marketplace listings priced above MSRP.
- B&H: free standard shipping over $49, NY sales tax applies (B&H is NY-based).
- Adorama (via Newegg listing): free shipping, NY sales tax applies.

**Recommended buying path for Pete (lowest total cost, single delivery):**

1. **store.ui.com**: order UCG-Fiber + U7 Pro together ($468 + shipping + NY tax). One package, full Ubiquiti warranty, no third-party Marketplace risk.
2. **Monoprice.com or Amazon**: order Cat6a patch cable assortment ($25 to $40, free shipping over Monoprice's threshold or with Prime).
3. **Skip the UACC-CM-RJ45-MG** unless Empire's install reveals a reason to want SFP+ WAN. Save the $65.

**Total at best prices:** ~$468 hardware + ~$30 cables + shipping/tax = **~$498 to $540 all-in** (without SFP+ adapter).

**If you do want to comparison-shop on the day of order:** check Newegg's [UCG-Fiber Adorama listing](https://www.newegg.com/p/0E6-003V-005X4). If Adorama still has it in stock at $279 with free shipping, that beats store.ui.com's price + shipping by $10 to $15. Same logic for U7 Pro on Newegg ([source](https://www.newegg.com/ubiquiti-networks-u7-pro/p/N82E16833664088)) at $189 with Newegg-direct free shipping.

**Reddit/community sentiment (May 2026):** No r/Ubiquiti deal threads turned up real discounts on either SKU. The community consistently reports buying directly from store.ui.com for warranty cleanliness and treating Newegg, Adorama, and B&H as backup options when Ubiquiti is out of stock. UCG-Fiber has been "in and out of stock" since launch ([source](https://www.broadbandbuyer.com/products/53336-ubiquiti-ucg-fiber/)). Set up a stock alert if it's unavailable when you check.

## Confidence Assessment

### High Confidence

- UCG-Fiber specs (1x 10G SFP+ + 1x 10GbE RJ45 WAN, 4x 2.5GbE + 1x 10GbE RJ45 + 2x 10G SFP+ LAN, 30W PoE budget, 5 Gbps IDS/IPS, $279) — confirmed at the official Ubiquiti tech specs page and store ([source](https://techspecs.ui.com/unifi/cloud-gateways/ucg-fiber)) ([source](https://store.ui.com/us/en/products/ucg-fiber))
- UACC-CM-RJ45-MG SFP+ to RJ45 adapter supports 1G/2.5G/5G/10G, $65 — confirmed on the official Ubiquiti store product page ([source](https://store.ui.com/us/en/products/uacc-cm-rj45-mg))
- UCG-Max specs (2.5GbE WAN, 4x 2.5GbE LAN, 1x 1GbE LAN, 2.3 Gbps IDS/IPS, $199, in stock) — confirmed at the official Ubiquiti tech specs page and store ([source](https://techspecs.ui.com/unifi/cloud-gateways/ucg-max)) ([source](https://store.ui.com/us/en/category/all-cloud-gateways))
- U7 Pro specs (Wi-Fi 7, 2.5GbE PoE+ uplink, 6 spatial streams, $189, in stock) — confirmed at official Ubiquiti tech specs ([source](https://techspecs.ui.com/unifi/wifi/u7-pro)) ([source](https://store.ui.com/us/en/category/all-wifi))
- Empire Access top tier is **2 Gbps symmetric**, not 5 Gbps — confirmed on Empire's own FAQ and press release ([source](https://www.empireaccess.com/faq/internet-speeds/)) ([source](https://www.empireaccess.com/empire-access-upgrades-residential-internet-upload-speeds-by-1000-percent/))
- UniFi multi-SSID + VLAN workflow — official Ubiquiti documentation ([source](https://help.ui.com/hc/en-us/articles/26136823938583-Creating-UniFi-WiFi-SSIDs))
- SSH is supported but Ubiquiti officially recommends GUI/Debug Console — official help center article ([source](https://help.ui.com/hc/en-us/articles/204909374-Connecting-to-UniFi-with-Debug-Tools-SSH))
- Cat6 vs Cat6a 2.5G/10G distance limits — multiple cabling vendor sources agree ([source](https://www.truecable.com/blogs/cable-academy/cat6-vs-cat6a))

### Medium Confidence

- **Empire Access lets you bring your own router.** Empire's docs say they "provide all the equipment" but don't explicitly say BYOR is allowed or denied. Other Calix-based fiber ISPs typically allow it because the ONT has a standard Ethernet handoff. Confirm with the installer.
- **Empire's installed equipment is a Calix Gigaspire combo.** Empire's marketing references the CommandIQ app, which is a Calix product, suggesting Calix Gigaspire hardware. Not directly confirmed in any Empire doc.
- **UDR7 LAN port count.** The store description mentions "10G" and "PoE switch" but doesn't list the exact port breakdown. Likely 4x 2.5G LAN + 1x 10G LAN based on the UDR-5G-Max sibling, but this needs verification on a physical unit or in the tech specs PDF.

### Low Confidence

- **Whether Empire offers a separate ONT-only install** vs always installing an integrated combo. Different Empire markets may have different equipment.

## Open Questions

Things Pete should verify with Empire on day one:

1. What's the exact ONT model being installed? (Calix model number on the box.)
2. Does the ONT have a separate Ethernet jack you can use directly, OR is it integrated into a Gigaspire combo unit?
3. If it's an integrated combo: can it be put in pure-bridge / IP-passthrough mode so the UCG-Max gets the public IP and handles routing, NAT, firewall?
4. Is there a monthly equipment fee for Empire's Wi-Fi router that goes away if you decline it?
5. Does declining the managed Wi-Fi (CommandIQ/ProtectIQ) affect support? (Most ISPs will troubleshoot up to the ONT/handoff and stop there.)

## Sources

### Ubiquiti / UniFi (official)
- [Ubiquiti store — Cloud Gateways category](https://store.ui.com/us/en/category/all-cloud-gateways)
- [Ubiquiti store — Wi-Fi APs category](https://store.ui.com/us/en/category/all-wifi)
- [UCG-Max product page](https://store.ui.com/us/en/products/ucg-max)
- [UCG-Fiber product page](https://store.ui.com/us/en/products/ucg-fiber)
- [UACC-CM-RJ45-MG SFP+ to RJ45 adapter product page](https://store.ui.com/us/en/products/uacc-cm-rj45-mg)
- [UDR7 product page](https://store.ui.com/us/en/products/udr7)
- [UX7 (Express 7) product page](https://store.ui.com/us/en/products/ux7)
- [U7 Pro product page](https://store.ui.com/us/en/products/u7-pro)
- [U7 Pro Wall product page](https://store.ui.com/us/en/products/u7-pro-wall)
- [U7-IW product page](https://store.ui.com/us/en/products/u7-iw)
- [UCG-Max tech specs](https://techspecs.ui.com/unifi/cloud-gateways/ucg-max)
- [UCG-Fiber tech specs](https://techspecs.ui.com/unifi/cloud-gateways/ucg-fiber)
- [UX7 tech specs](https://techspecs.ui.com/unifi/cloud-gateways/ux7)
- [U7 Pro tech specs](https://techspecs.ui.com/unifi/wifi/u7-pro)
- [U7 Pro XG tech specs](https://techspecs.ui.com/unifi/wifi/u7-pro-xg)
- [U7 Lite tech specs](https://techspecs.ui.com/unifi/wifi/u7-lite)
- [U7 In-Wall tech specs](https://techspecs.ui.com/unifi/wifi/u7-iw)
- [Connecting to UniFi with Debug Tools & SSH (official)](https://help.ui.com/hc/en-us/articles/204909374-Connecting-to-UniFi-with-Debug-Tools-SSH)
- [Creating UniFi Wi-Fi SSIDs (official)](https://help.ui.com/hc/en-us/articles/26136823938583-Creating-UniFi-WiFi-SSIDs)
- [Creating Virtual Networks (VLANs) (official)](https://help.ui.com/hc/en-us/articles/9761080275607-Creating-Virtual-Networks-VLANs)

### Empire Access (official)
- [Empire Fiber Internet residential plans](https://www.empireaccess.com/residential/high-speed-fiber-internet/)
- [Empire Fiber Internet equipment FAQ](https://www.empireaccess.com/faq/what-equipment-do-i-need-for-empire-fiber-internets-internet-service/)
- [Empire Fiber Internet speeds FAQ](https://www.empireaccess.com/faq/internet-speeds/)
- [Empire Fiber Internet upload speed upgrade announcement (March 2024)](https://www.empireaccess.com/empire-access-upgrades-residential-internet-upload-speeds-by-1000-percent/)
- [Empire Fiber Internet how-it-works/install](https://www.empireaccess.com/understanding-fiber-internet-and-how-its-installed/)
- [Empire Fiber Internet communities served](https://www.empireaccess.com/about-us/communities-we-serve/)

### Cabling
- [TrueCable — Cat6 vs Cat6a](https://www.truecable.com/blogs/cable-academy/cat6-vs-cat6a)

### Third-party UniFi guides (used for verification)
- [ITMan — Setting up secure guest and IoT networks on UniFi APs](https://itman.ae/2025/07/07/set-up-secure-guest-iot-network-on-unifi-aps/)
- [LazyAdmin — UniFi VLAN configuration](https://lazyadmin.nl/home-network/unifi-vlan-configuration/)
- [LazyAdmin — UniFi SSH commands](https://lazyadmin.nl/home-network/unifi-ssh-commands/)
- [Avative Internet — BYOR + ONT (Calix-based ISP, used as reference for typical handoff)](https://support.avative.com/hc/en-us/articles/40119741588116-WiFi-Routers-Mesh-Extenders)

### Retail price verification (May 2026)
- [Newegg — UCG-Fiber listing (Adorama at $279, free shipping)](https://www.newegg.com/p/0E6-003V-005X4)
- [Newegg — U7 Pro listing ($189, Newegg-shipped)](https://www.newegg.com/ubiquiti-networks-u7-pro/p/N82E16833664088)
- [B&H Photo — UCG-Fiber listing](https://www.bhphotovideo.com/c/product/1884945-REG/ubiquiti_networks_ucg_fiber_cloud_gateway_fiber.html)
- [B&H Photo — U7 Pro listing](https://www.bhphotovideo.com/c/product/1805878-REG/ubiquiti_networks_u7_pro_us_unifi_wifi_7_pro.html)
- [Broadband Buyer — UCG-Fiber stock-availability commentary](https://www.broadbandbuyer.com/products/53336-ubiquiti-ucg-fiber/)

## How This Report Was Generated

Web research May 1, 2026 ET. SearXNG MCP was unavailable (timeouts on all queries), so primary sources were direct WebFetch on store.ui.com and techspecs.ui.com product/spec pages, supplemented by built-in WebSearch for Empire Access and third-party UniFi/cabling references. Every product spec was pulled from Ubiquiti's official tech specs site rather than third-party reviews. The "5 Gbps" claim in the original prompt was checked against Empire's own FAQ and corrected: Empire residential tops out at 2 Gbps as of May 2026.

## Update History

- **2026-05-01 21:25 ET** — Added "Where to Buy / Best Deals" section after Recommended BOM. Verified all four BOM items at store.ui.com baseline prices (UCG-Fiber $279, U7 Pro $189, UACC-CM-RJ45-MG $65) and cross-checked third-party retailers (Newegg, Adorama, B&H, Amazon). Finding: Ubiquiti products do not discount meaningfully through third-party channels. store.ui.com is the cheapest source for the gateway and AP at MSRP; Newegg/Adorama match price and offer free shipping but rarely undercut. No public "UI Store Refurbished" or "Early Access" pricing applies to UCG-Fiber or U7 Pro as of May 2026. Cables are cheaper at Monoprice or Amazon than at store.ui.com. Recommended buying path: gateway + AP from store.ui.com in one order, cables from Monoprice or Amazon Prime, skip the SFP+ adapter unless install reveals a reason.
- **2026-05-01 21:11 ET** — Swapped UCG-Max for UCG-Fiber as the recommended gateway. UCG-Fiber adds a 10G SFP+ WAN port, a 10GbE RJ45 WAN port, an integrated 30W PoE budget (powers the U7 Pro AP directly with no separate adapter), 5 Gbps IDS/IPS (vs UCG-Max's 2.3 Gbps), and 7 LAN ports total including 10G options. Empire's RJ45 ONT handoff plugs directly into the 10GbE RJ45 WAN port — no transceiver required. Optional UACC-CM-RJ45-MG SFP+ to RJ45 adapter ($65) added as a line item for users who want to wire WAN through the SFP+ port instead. Hardware total moved from ~$433 to ~$498 (or ~$563 with the SFP+ adapter). Updated TL;DR, Current Status, Section 2 (Gateway Selection), Section 3 (AP powering note), Section 4 (port math), Section 8 (setup steps), Recommended Bill of Materials, Confidence Assessment, and Sources.
