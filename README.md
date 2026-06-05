# HCFA — Hybrid Copper-Fiber Architecture

**One Cable to the Ceiling**

> Fiber data and PoE power in a single run, using components already on your shelf

**Series:** Hybrid Copper-Fiber Architecture · June 2026  
**Related:** [OCFA — Open Consumer Fiber Architecture](https://github.com/[your-handle]/ocfa-spec)

---

If you are already running separate fiber and copper to high-speed access points, or deploying BiDi SFP+ connections in your infrastructure — **you are already doing most of what this spec describes.**

**HCFA — the Hybrid Copper-Fiber Architecture** — takes the approach experienced network engineers already use for high-speed AP and remote switch installations, names it, standardises the port layout and cable construction, and specifies the power delivery and resilience behaviour clearly enough that any manufacturer can build to it and any installer can deploy it. The standards it uses are unchanged. The components are the same ones already on the shelf. The only thing that changes is that it becomes a defined, repeatable process rather than something each engineer figures out independently.

It builds on [OCFA](https://github.com/[your-handle]/ocfa-spec), inheriting its BiDi port and wavelength convention. The short version: OCFA standardises 10G BiDi fiber edge connections with factory-fixed polarity so the TX/RX error is physically impossible. HCFA applies that port standard to getting fiber data and PoE power to ceiling devices and remote switch cabinets in a single cable run.

> **Existing infrastructure:** if you already have separate fiber and copper runs to ceiling APs or remote cabinets, both cables plug directly into the HCFA hybrid edge port — fiber into the LC socket, copper into the RJ45. No rewiring needed. The composite cable is the better option for new runs only.

---

## The Problem: Two Cables Where One Should Do

Most Wi-Fi access points still run on a single copper cable — Cat5e or Cat6 carrying both data and PoE power, and that works fine. But high-speed APs are hitting the ceiling of what copper can deliver at real building distances, which is why the top-end units from Ubiquiti and others now ship with an SFP+ port alongside the RJ45 — fiber for data, copper for power.

If you've deployed one of those, you've done this: pulled two cables. Two conduit entries, two terminations, two patch leads at the switch end — and a field-insertable SFP+ transceiver that adds cost and a polarity problem to every installation. It works, but it's double the labour and double the things that can go wrong.

HCFA replaces that for new runs while remaining fully compatible with every two-cable installation already in place.

---

## The Hybrid Edge Port

The core of HCFA is a switch port with two sockets on the same faceplate:

- **Upper: LC optical socket** — Port-A (TX 1310nm / RX 1550nm), fixed integrated BiDi optics, 10G
- **Lower: RJ45 socket** — PoE, at least PoE++ rated per port independently

That per-port requirement matters. Many PoE switches have a total power budget spread across all ports. HCFA requires that every port can deliver its full rated wattage simultaneously — a 16-port switch can run 16 cameras at full power without anything getting throttled.

The optical port carries 10G data. The copper port carries power and a 1G management link on a dedicated VLAN — always separate from user data, always reachable. The switch presents both as a single logical port in the management interface — no separate configuration required.

Both connectors are standard off-the-shelf components. No proprietary connectors, no modified tooling.

---

## Two Cables or One — Your Choice

Existing two-cable installations (separate fiber and copper already in the walls) connect directly to the hybrid edge port with no changes. One cable into the LC socket, one into the RJ45. The composite cable is purely an improvement for new runs — it doesn't obsolete anything already installed.

---

## The Composite Cable

For new installations, HCFA defines a **Cat6+fiber composite cable**: standard Cat6 construction with a single fiber strand. Three valid construction approaches:

**Option A — fiber strand added inside standard jacket.** Minimal size increase. Standard boot manages the fiber exit at termination.

**Option B — buffered fiber inside jacket.** Around 110% of standard Cat6 diameter. Easier field termination, better for general installers.

**Option C — fiber replaces the Cat6 central spline (recommended).** Standard Cat6 has a central cross-filler whose only job is to keep the pairs separated — no signal, no electrical function. A 250-micron coated fiber occupies that position instead. The pairs wrap around it as normal. The cable is **identical in diameter to standard Cat6**, the fiber is fully protected throughout the run, and at termination it emerges from the centre naturally ready for LC termination. Best protection, smallest footprint, cleanest termination.

At each end: fiber terminates to LC connector, copper pairs terminate to RJ45. Standard wall plates, standard keystones, standard patch panels.

### Supply Formats

**Pre-terminated lengths up to 100 m** — factory-fitted boots, LC connectors, and RJ45 at both ends. No optical tooling required on site. 100 m covers the vast majority of AP and remote switch runs.

**Drum cable with boots supplied separately** — for installers with fiber termination equipment. Standard RJ45 crimping for the copper, standard LC termination or fusion splice for the fiber. Nothing proprietary.

> A fusion splicer costs upwards of £1,000. Pre-terminated to 100 m removes that barrier entirely — the job becomes cable routing, nothing more.

The copper element runs at **1 Gbps by default** — reliable at full Cat6 distance. The PoE standard is deliberately open: HCFA does not fix a specific IEEE revision, so higher-wattage future standards are accommodated without requiring a spec change.

---

## Remote Switches and APs — Same Ports, Mixed on One Switch

From the core switch's perspective, a hybrid port serving an AP and one serving a remote switch are identical — both are Port-A (TX1310/RX1550) connecting to a Port-B (TX1550/RX1310) device. The remote switch's uplink port mirrors the core hybrid port: Port-B (TX1550/RX1310) optical socket plus RJ45 PoE input. Blue bezel on the core, green bezel on the remote switch uplink — one glance confirms correct wiring.

A single core HCFA switch can serve APs, remote switches, and direct-connected devices simultaneously with no special configuration. Swapping an AP for a remote switch on any port is a physical reconnection — only the PoE budget needs checking, since a remote switch powering multiple cameras draws significantly more than a single AP.

---

## Remote Switches: Powered by the Cable, Scales When Needed

A single composite cable run carries enough PoE to power the remote switch and its connected cameras. Power mode is determined automatically by whether a local PSU is connected.

### PoE-Only Mode (No Local PSU)

Total available downstream budget = **incoming PoE wattage − switch chassis overhead (~10–15W)**

At 90W incoming (PoE++, 802.3bt Type 4), approximately 75–80W feeds downstream devices. Fixed cameras drawing 4–11W each — a well-specified 8-port deployment is comfortably achievable on one cable run with no local power at all.

PTZ cameras (typically 15–25W when active) consume headroom quickly. A cabinet with PTZ units should use an external PSU.

### External PSU Mode

Downstream ports unlock **full PoE++ per port independently**. PTZ cameras, high-draw APs, anything power-hungry — all fully supported. The incoming PoE from the composite cable continues to power the switch chassis; the PSU's full output goes to downstream devices.

Connecting the PSU while the switch is running upgrades power gracefully — no ports drop, no resets. A physical LED on the chassis shows the active mode without needing to log in.

The PoE standard is open — as IEEE continues to evolve higher-wattage standards, the switch design accommodates them without a spec change.

---

## Fiber Fault Resilience

| Condition | Optical (LC) | Copper (RJ45) | Result |
|---|---|---|---|
| Normal | Active | Active | 10G data + PoE power |
| Fiber fault | Down | Active | Auto-failover: 1G copper + PoE power |
| Power fault | Active | Down | Optical data only, self-powered device |

On fiber loss, the switch activates copper-only mode automatically. The remote switch stays powered. It stays reachable at 1G over the copper management path. Cameras keep recording. No site visit needed — the fault can be diagnosed and recovery coordinated remotely. Restore the fiber and full 10G operation returns automatically.

---

## Port Colour Coding (Optional, Recommended)

| Colour | Port | Role |
|---|---|---|
| Blue | Port-A (TX1310/RX1550) LC | Downlink — APs, remote switches, devices |
| Green | Port-B (TX1550/RX1310) LC | Uplink — connects this switch upstream |
| Amber | SFP+ cage | Infrastructure uplink |
| White/Grey | RJ45 PoE | Copper power and management |

Plastic bezel colouring is a pigment in the injection mould — zero additional cost at scale. Does not apply to laptop NICs or adapters.

---

## Management: No New Protocols Needed

The copper sub-interface is a standard 1G Ethernet link on a management VLAN. UniFi's adoption protocol works over it. LLDP works over it. MikroTik's discovery works over it. SNMP works over it.

A UniFi deployment stays a UniFi deployment. A MikroTik network stays a MikroTik network. HCFA adds the physical port and cable architecture — it doesn't touch the management stack.

---

## Summary

| Scenario | Result |
|---|---|
| Existing two-cable building | Both plug into hybrid port. 10G data and resilient management immediately. Zero rewiring. |
| New AP installation | One cable pull, one conduit entry, one termination. 10G + PoE++ + auto-failover. |
| Remote switch / CCTV cabinet | 10G uplink, switch powered by the cable, cameras powered by the switch, management retained on fiber fault. |

All using LC connectors and Cat6 that any installer already knows, managed by tools already in use.

---

## The Specification

The full technical specification is in **[HCFA-SPEC.md](./HCFA-SPEC.md)**, released under CC0 — no rights reserved. Build it.

Open contribution items currently needing input:
- Hybrid port faceplate mechanical dimensions (co-located LC + RJ45 bezel)
- Bifurcated termination boot mechanical design for cable construction Options A and B
- Remote switch PoE power budget calculations for common camera configurations
- Interoperability testing with UniFi/MikroTik/Cisco management protocols

---

## Licence

Released under [Creative Commons CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/) — no rights reserved. Build it.

---

*HCFA Draft 0.1 · June 2026*
