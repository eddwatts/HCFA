# HCFA: Hybrid Copper-Fiber Architecture
## Specification — Draft 0.1

**Status:** Draft  
**Author:** Open Infrastructure Systems Architect  
**Date:** June 2026  
**Repository:** https://github.com/[your-handle]/hcfa-spec  
**Related:** [OCFA — Open Consumer Fiber Architecture](./OCFA-SPEC.md)

---

## What Is HCFA?

**If you are already running BiDi SFP+ connections to your access points or remote switches, you are close to what this spec describes.**

The hybrid edge port — an LC optical socket and an RJ45 PoE socket on the same faceplate — is the productised version of what many network engineers already do with separate fiber and copper runs. HCFA names it, standardises the port layout, defines the cable construction, and specifies the power and resilience behaviour so that any manufacturer can build to it interoperably.

HCFA is an open standard for deploying high-speed fiber data and copper power delivery to edge infrastructure — Wi-Fi access points, remote switches, CCTV cameras, and any powered device that benefits from both a fast optical uplink and a resilient copper fallback — over a single cable run.

HCFA borrows its port and wavelength convention directly from [OCFA](./OCFA-SPEC.md). The BiDi Port-A/Port-B standard defined in OCFA applies unchanged to all HCFA optical connections. HCFA extends this by adding:

- A **hybrid edge port** combining an LC optical socket and an RJ45 PoE socket on a single faceplate
- A **Cat6+fiber composite cable** for new structured runs — one pull, one conduit entry, one termination
- A **remote switch profile** covering power delivery, management, and automatic fiber fault resilience
- Compatibility with **existing two-cable infrastructure** — no requirement to re-pull cables already in place

> **Existing infrastructure users:** If you already have separate fiber and copper runs to ceiling APs or remote cabinets, both cables plug directly into the HCFA hybrid edge port — one into the LC socket, one into the RJ45. No rewiring needed. The composite cable is purely the better option for new runs.

---

## Status of This Document

This is a working draft. Sections marked **[TBD]** are open for community contribution. Implementors SHOULD track the canonical repository for updates.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

---

## Table of Contents

1. [Port Convention — Inherited from OCFA](#1-port-convention--inherited-from-ocfa)
2. [The Hybrid Edge Port](#2-the-hybrid-edge-port)
3. [Cabling](#3-cabling)
4. [HCFA Switch Requirements](#4-hcfa-switch-requirements)
5. [Remote Switch Profile](#5-remote-switch-profile)
6. [Deployment Profiles](#6-deployment-profiles)
7. [Conformance](#7-conformance)

---

## 1. Port Convention — Inherited from OCFA

HCFA uses the OCFA BiDi port convention unchanged.

- **Port-A (TX 1310nm / RX 1550nm)** — switch/host side, blue marking
- **Port-B (TX 1550nm / RX 1310nm)** — device/client side, green marking

All HCFA optical connections MUST follow this convention. A correctly wired link connects one Port-A to one Port-B. If the link is up, it is correctly wired.

**Important:** existing BiDi SFP+ products use inconsistent A/B labelling with no universal industry standard. OCFA and HCFA compliance is determined by wavelength assignment, not by letter designation. The wavelength pair MUST always be shown alongside the port letter in markings, management interfaces, and documentation. Refer to [OCFA Section 2](./OCFA-SPEC.md) for the full compliance definition.

---

## 2. The Hybrid Edge Port

### 2.1 Purpose

The hybrid edge port provides both a 10G optical data connection and a copper power and management connection from a single switch port location, using two co-located standard connectors on a shared faceplate.

### 2.2 Physical Layout

- **Upper aperture:** LC/UPC optical socket — OCFA Port-A, fixed integrated BiDi optics
- **Lower aperture:** RJ45 socket — PoE to the current supported standard (see Section 4.2)

Both connectors MUST be unmodified, off-the-shelf components. No proprietary connector or modified tooling is used.

### 2.3 Two-Cable Compatibility

The hybrid edge port MUST accept traditional two-cable installations without modification:

- An existing separate fiber patch cord plugs into the LC upper aperture
- An existing separate copper patch cord plugs into the RJ45 lower aperture

This is functionally identical to a new composite cable installation. Existing infrastructure is not disturbed.

### 2.4 Operational Modes

Firmware MUST detect and handle all connection states automatically with no manual configuration:

| State | Optical (LC) | Copper (RJ45) | Data Path | Power |
|---|---|---|---|---|
| Full hybrid | Connected | Connected | 10G optical | PoE copper |
| Fiber-only | Connected | Not connected | 10G optical | None |
| Copper-only | Not connected | Connected | 1G copper | PoE copper |
| Unpopulated | Not connected | Not connected | None | None |

### 2.5 Copper Sub-Interface

In full hybrid mode the copper sub-interface operates at **1 Gbps** and SHOULD be assigned to a dedicated management VLAN, providing out-of-band management access and a resilient fallback data path independent of the optical link.

### 2.6 Logical Interface Model

The hybrid edge port MUST be presented in the management interface as a **single logical port** with two sub-interfaces. Administrators MUST NOT be required to configure the optical and copper connections as separate ports.

### 2.7 Recommended Port Colour Scheme

Consistent port colouring across HCFA hardware allows installers to identify port roles instantly without reading labels. Colour coding is OPTIONAL but strongly RECOMMENDED for all OCFA and HCFA optical and hybrid ports on switch hardware. It does not apply to laptop NICs, USB-C adapters, or PCIe cards.

| Colour | Port Type | Role |
|---|---|---|
| **Blue** | Port-A (TX1310/RX1550) LC socket | Downlink — device, AP, or remote switch connection |
| **Green** | Port-B (TX1550/RX1310) LC socket | Uplink — to upstream OCFA/HCFA switch |
| **Amber** | SFP+ uplink cage | Infrastructure uplink — long-haul or high-speed core |
| **White / Light grey** | RJ45 PoE socket | Copper power and management |

A glance at a switch faceplate immediately communicates: blue ports connect to devices, green port connects upstream, amber cage connects to core infrastructure. Plastic bezel colouring is zero additional cost at manufacturing scale.

### 2.8 Device Discovery and Management

The copper sub-interface MUST be presented as a standard **IEEE 802.3 Ethernet interface**. No HCFA-specific discovery or adoption protocol is defined or required.

Device discovery, management, and adoption SHOULD use the switch platform's existing management protocols over this interface — LLDP, UniFi adoption, MikroTik discovery, SNMP, or equivalent. The copper sub-interface on a management VLAN is indistinguishable from any other management Ethernet port to the software layer. This means:

- Existing managed devices (APs, cameras, phones) that already communicate over a management VLAN require no firmware changes to work with HCFA
- Operators retain their existing management tooling and ecosystem — a UniFi deployment stays a UniFi deployment
- No conformance burden is introduced beyond passing standard Ethernet traffic on the management VLAN

---

## 3. Cabling

### 3.1 Traditional Two-Cable Installation (Existing Infrastructure)

Where separate fiber and copper runs already exist — installed independently at any prior time — both cables connect to the hybrid edge port as described in Section 2.3. No HCFA-specific cable is required.

### 3.2 Cat6+Fiber Composite Cable (New Runs)

For new structured runs that terminate at a hybrid edge port, a composite cable MAY be used in place of two separate cables. This delivers both the optical and copper elements in a single pull.

The composite cable MUST contain:
- A Cat6 copper element meeting TIA-568-C.2 (or equivalent) for current carrying capacity and copper data transmission
- 1× OM4 or OM5 single-strand fiber running inside the same outer jacket

Construction follows standard Cat6 jacket and bend radius conventions, allowing the cable to be handled using existing structured cabling tooling and practices.

**This cable type is for new runs terminating at hybrid edge ports only.** It is not a general infrastructure replacement.

#### 3.2.1 Copper Speed

The copper element operates at **1 Gbps (1000BASE-T) by default.** These are longer structured runs — potentially up to 100 m — where 1G over Cat6 is fully reliable across the full distance. 2.5GBASE-T is OPTIONAL where run length and cable quality permit but MUST NOT be assumed as a baseline.

#### 3.2.2 PoE Standard

The composite cable copper element MUST support the PoE standard claimed by the implementation. HCFA does not mandate a specific IEEE PoE revision — the standard continues to evolve and higher-wattage variants SHOULD be adopted as they are ratified. Conductor gauge and jacket construction MUST be sufficient for the wattage class the product claims. Implementations MUST clearly state the supported PoE standard and maximum wattage in product documentation.

At time of writing, **IEEE 802.3bt Type 4 (PoE++, 90W)** is the RECOMMENDED baseline for AP and remote switch applications.

#### 3.2.3 Cable Construction Options

HCFA does not mandate a specific internal construction for the composite cable. Manufacturers MAY choose from the following approaches based on their production processes and the target market. All three produce a cable that meets the functional requirements of this specification.

**Option A — Fiber strand added inside standard Cat6 jacket**
A bare or lightly coated fiber strand is laid alongside the four twisted pairs inside the standard Cat6 outer jacket. Cable diameter increases minimally. At termination the fiber emerges from the jacket and requires careful handling over its exposed length. A protective sleeve or standard loose-tube boot manages the exit point. Smallest cable profile.

**Option B — Buffered fiber added inside jacket**
The fiber strand carries its own 900-micron tight buffer before entering the jacket. Cable diameter increases to approximately 110% of standard Cat6. The buffered fiber handles more robustly at termination and is easier for field termination. Preferred where regular field termination by general installers is expected.

**Option C — Fiber replaces the Cat6 central spline**
Standard Cat6 cable includes a central spline or cross-filler whose sole function is to maintain pair geometry. This spline carries no signal and has no electrical function. In Option C, a coated fiber strand (typically 250-micron) occupies this central position in place of the spline. The four twisted pairs wrap around the fiber as they would around the spline. The outer jacket is applied as normal.

This produces a cable with **identical outer diameter to standard Cat6**. The fiber is fully protected on all sides by the surrounding pairs and outer jacket throughout the cable run. At termination, stripping the outer jacket causes the fiber to emerge naturally from the centre of the cable with its coating intact, ready for LC termination. Standard cable boots manage the transition. This is the RECOMMENDED construction where a manufacturer can support it, as it provides the best protection, smallest footprint, and cleanest termination behaviour.

#### 3.2.4 Termination and Boot

At each end the fiber terminates to an **LC/UPC connector** and the copper pairs terminate to a standard **RJ45 plug**. Both use standard industry tooling.

The boot or strain relief design at the termination point follows naturally from the cable construction chosen. Boot designs for all three construction options have direct precedents in existing hybrid cable products (broadcast, AV, and armoured composite cables). No novel boot design is required — the manufacturer selects an appropriate boot for their construction. The functional requirement is that the fiber exit angle MUST NOT exceed the fiber strand's minimum bend radius under normal installation and plug/unplug forces.

#### 3.2.5 Supply Formats

Composite cable SHOULD be made available in the following formats to suit different installer capabilities:

**Pre-terminated lengths**
Factory-fitted bifurcated boots and connectors at both ends. Available in standard lengths up to and including **100 m**. Recommended standard lengths: 1 m, 2 m, 3 m, 5 m, 10 m, 15 m, 20 m, 30 m, 50 m, 75 m, 100 m.

Pre-terminated cable requires no optical termination equipment on site. An installer needs only the ability to route and pull cable. This format is the RECOMMENDED choice for most AP and remote switch deployments, as 100 m covers the overwhelming majority of single-run scenarios without requiring field termination.

**Drum (bulk) with boots supplied**
Cable supplied on a drum by the metre, with bifurcated boots supplied separately per-termination. The installer terminates both ends using standard **RJ45 crimping tooling** for the copper element and standard **LC fiber termination or fusion splice equipment** for the optical strand. No proprietary tooling is required beyond what a competent network and fiber engineer already carries.

Boots SHOULD also be available to purchase separately from the cable drum, so installers with existing cable stock can order boots independently.

This format is suited to installers who perform regular fiber termination work and have the optical tooling to justify. For occasional or small-volume runs, pre-terminated lengths are more practical.

---

## 4. HCFA Switch Requirements

An HCFA-compliant switch implements the OCFA port standard on its downlink ports and adds the hybrid edge port and PoE delivery defined in this specification.

### 4.1 Downlink Ports

All HCFA switch downlink ports MUST implement the OCFA Port-A specification — fixed BiDi LC optics, 1310 nm TX / 1550 nm RX, LC/UPC connector, no SFP cage.

### 4.2 PoE Per Port

Each RJ45 socket on an HCFA switch MUST support the full rated PoE wattage **per port independently.** A shared or budgeted PoE allocation across multiple ports is NOT sufficient for HCFA conformance. Each port must be capable of delivering its full rated wattage simultaneously with all other ports at full load.

The minimum RECOMMENDED per-port PoE standard is **IEEE 802.3bt Type 4 (PoE++, 90W)**. Higher standards SHOULD be adopted as they are ratified.

### 4.3 Uplink Ports

HCFA switches MUST provide the same uplink options as OCFA switches:
- At least one fixed **Port-B uplink** for zero-configuration OCFA/HCFA switch daisy-chaining
- At least one open **SFP+ cage** for long-haul or high-speed uplinks to existing infrastructure

### 4.4 Automatic Failover

Firmware MUST monitor optical link state and transition to copper-only mode automatically on fiber link loss, without requiring administrator action. Restoration of the fiber link MUST return full hybrid operation automatically.

---

## 5. Remote Switch Profile

### 5.1 Overview

A remote HCFA switch is a compact switch deployed at a satellite location — CCTV cabinet, floor distribution point, external structure — connected to the core switch via a single composite cable run or two traditional cable runs. The fiber strand carries 10G production data; the copper element powers the switch and provides a 1G management and fallback path.

### 5.2 Remote Switch Uplink Port

The remote switch uplink port MUST be a hybrid edge port as defined in Section 2, presenting:

- **LC socket — Port-B (TX 1550nm / RX 1310nm)** — optical uplink to the core switch Port-A (TX 1310nm / RX 1550nm)
- **RJ45 socket** — incoming PoE power feed and 1G management path

This is the mirror image of the hybrid edge port on the core switch. The composite cable connects Port-A on the core to Port-B on the remote switch — one end blue bezel, one end green bezel. If the link is up, it is correctly wired.

### 5.3 Mixed Deployment — APs and Remote Switches on the Same Core Switch

From the core switch's perspective, a hybrid edge port serving an AP and a hybrid edge port serving a remote switch are identical — both are Port-A (TX1310/RX1550) optical sockets connecting to a Port-B (TX1550/RX1310) device. The core switch requires no special configuration to serve a mix of APs, remote switches, and direct-connected devices simultaneously on the same switch fabric.

Any hybrid port on a core HCFA switch MAY serve:
- A Wi-Fi access point
- A remote HCFA switch
- Any other HCFA-compatible Port-B device

Switching between device types on a given port requires only physically reconnecting the cable. The only operational consideration when changing from an AP to a remote switch (or vice versa) is the PoE power budget — a remote switch powering multiple cameras draws significantly more than a single AP and the upstream port budget MUST be verified before switching device types.

### 5.2 Power

Remote HCFA switches operate in one of two power modes, determined automatically by whether a supplemental external power supply is connected.

#### 5.2.1 PoE-Only Mode (Single Cable, No External PSU)

When powered solely from the incoming PoE feed on the composite cable copper element, the total available downstream budget is:

**Incoming PoE wattage − Switch chassis overhead = Available downstream budget**

The switch MUST publish its chassis power draw in product documentation so installers can calculate remaining downstream headroom. In this mode, downstream PoE ports operate at standard PoE levels, shared across the available budget.

At IEEE 802.3bt Type 4 (90W) incoming and a typical 10–15W chassis draw, approximately 75–80W remains for downstream devices. Fixed cameras drawing 4–11W each mean a well-specified 8-port deployment is comfortably achievable on a single cable run.

**PTZ cameras and high-draw devices** (typically 15–25W or more when active) consume headroom quickly in PoE-only mode. Deployments including PTZ units SHOULD use an external PSU unless budget calculations confirm adequate margin.

The switch MUST NOT exceed the available budget. Ports SHOULD negotiate power allocation and decline to power devices that would cause an overrun rather than dropping already-powered devices unexpectedly.

#### 5.2.2 External PSU Mode (Supplemental Power Connected)

When a supplemental external power supply is connected, downstream ports unlock **full PoE++ per port independently** (IEEE 802.3bt Type 4 minimum; higher standards as ratified). The incoming PoE feed from the composite cable continues to power the switch chassis, freeing the full external PSU budget for downstream ports. PTZ cameras, high-draw APs, and other high-wattage devices are fully supported in this mode.

As with the incoming PoE standard, HCFA does not fix the external PSU PoE revision — implementations SHOULD adopt current IEEE PoE standards at time of manufacture.

#### 5.2.3 Mode Transition

Switching between modes MUST be automatic and non-disruptive. Connecting an external PSU while the switch is running MUST NOT cause existing powered ports to drop or reset. Additional budget becomes available to new port negotiations without disturbing active connections.

#### 5.2.4 Power Mode Indicator

The switch MUST provide a visible indicator — LED or equivalent on the chassis — showing the current power mode. Installers MUST be able to determine the active mode without accessing the management interface.

### 5.3 Normal Operation

| Path | Medium | Function |
|---|---|---|
| Optical (LC) | Fiber strand | 10G production data — all device traffic |
| Copper (RJ45) | Cat6 element | PoE power feed + 1G management VLAN |

### 5.4 Fiber Fault Resilience

On optical link loss, the HCFA hybrid port activates copper-only mode automatically:

- The remote switch remains powered from the copper PoE feed
- Management traffic continues at 1 Gbps over the copper data path
- Connected downstream devices remain operational
- The management system retains connectivity for diagnostics and recovery

No manual intervention is required to fail over or restore.

### 5.5 Typical Application: CCTV / Camera Cabinet

A 16-port PoE remote switch in a camera cabinet connects to the core via a single composite cable run. The upstream PoE budget powers the remote switch. The remote switch's downstream PoE ports power the cameras. All camera streams travel to the core at 10G over the fiber. If the fiber is damaged or dirty, the copper path keeps the cabinet reachable and cameras remain powered and recording.

---

## 6. Deployment Profiles

### 6.1 Existing Infrastructure — No Rewiring

Buildings with separate fiber and copper runs already in place connect both to the HCFA hybrid edge port. Data transitions to 10G optical; copper continues as the power and management path. Zero structural change.

### 6.2 New AP Installation

Single composite cable pull from comms cabinet to ceiling mounting point. Terminates to LC keystone (fiber) and RJ45 keystone (copper) at the AP end; connects to hybrid edge port at the switch end. AP carries fixed Port-B optics. One pull, full hybrid capability, PoE++ power, 10G data.

### 6.3 Remote Switch / CCTV Cabinet

Single composite cable pull to the remote cabinet. Remote HCFA switch powered by incoming PoE. Downstream PoE ports serve cameras or other devices. 10G optical uplink to core. Automatic failover to 1G copper management on fiber fault.

### 6.4 Mixed Deployment

Existing copper-only legacy devices connect to the RJ45 port and operate at 1G. New OCFA-NIC devices and HCFA APs connect optically at 10G. Both coexist on the same switch with no configuration changes.

---

## 7. Conformance

A device claiming **HCFA compliance** MUST implement:

- Section 1 — OCFA Port-A/Port-B BiDi wavelength convention
- Section 2.2–2.4 — hybrid edge port layout, two-cable compatibility, and automatic mode detection (switch products)
- Section 2.6 — unified logical port model (switch products)
- Section 2.8 — device discovery using existing platform management protocols (switch products)
- Section 4.2 — full rated PoE per port independently (switch products)
- Section 4.4 — automatic fiber fault failover (switch products)
- Section 5.2–5.4 — remote switch power and resilience behaviour (remote switch products)

Sections 3.2, 3.2.4, and 5.5 are OPTIONAL for a conformance claim but RECOMMENDED.

---

## Contributing

Priority areas for community input:

- [ ] **Bifurcated termination boot** — mechanical design for Cat6+fiber split termination (Section 3.2.4)
- [ ] Hybrid port faceplate mechanical dimensions (co-located LC + RJ45 bezel)
- [ ] Remote switch PoE power budget calculations for common camera and AP configurations
- [ ] Per-port PoE budgeting guidance for high-density remote switch deployments
- [ ] Interoperability with UniFi/MikroTik/Cisco management protocols

---

## Related Standards

**[OCFA — Open Consumer Fiber Architecture](./OCFA-SPEC.md)** defines the BiDi port convention, edge NIC, patch cord, and switch port standard that HCFA builds upon.

---

## Licence

Released under [Creative Commons CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/) — no rights reserved. Build it.
