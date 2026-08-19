# Zones and Conduits

**Document:** `docs/01-design/02-zones-and-conduits.md`
**Phase:** 1 — Design
**Status:** Baseline
**Depends on:** `01-asset-inventory.md`

---

## 1. Purpose and method

This document partitions the assets identified in the inventory into security zones and defines every permitted flow between them as a conduit. It is the controlling design document for the project: the addressing plan, the virtual local area network (VLAN) allocation, the access control lists applied in Packet Tracer and the firewall policy all derive from what is written here. Where a later document contradicts this one, this one is authoritative until it is formally revised.

### 1.1 Why zones rather than Purdue levels

The Purdue Enterprise Reference Architecture (PERA) describes a hierarchy of functions, from field devices at Level 0 to enterprise systems at Level 4. It was developed to describe data flow in automated plants and was not originally a security model. IEC 62443 adopted its vocabulary as a starting point but defines segmentation differently: a zone is a grouping of assets that share a common protection requirement, established by risk assessment rather than by function.

The practical consequence is that zone boundaries and Purdue levels do not align neatly. This design contains three cases where they diverge, and each divergence is deliberate:

- **Zone 4 (Optimisation)** is separated from Zone 3 (Sawmill process control) although both sit at Purdue Levels 1 and 2. The scanner optimiser consoles are general-purpose computers with commodity operating systems and relational databases operating inside a real-time control path. They require patching cycles like an information technology (IT) asset and uptime like a control asset. Those two requirements are in direct conflict, which means the assets do not share a protection requirement with the programmable logic controllers (PLC) they serve, and therefore do not belong in the same zone.

- **Zone 8 (Weighbridge and gatehouse)** spans Purdue Levels 1 through 4 in a single small zone. Its hardware is Level 1: load cells and a controller. Its data is Level 3 and Level 4: the first record in the chain of custody, and the basis for goods receipt and invoicing. It sits physically at the site perimeter, in an unstaffed structure, outside the plant buildings. No existing zone can absorb it without importing that exposure.

- **Zone 10 (Site infrastructure)** is formally a Level 3 grouping, but it is assigned a target security level equal to the process control zones. Industrial incident data consistently shows that ransomware operators rarely manipulate control systems directly; production stops because the identity, virtualisation and enterprise systems that production depends upon are encrypted or precautionarily shut down. Treating identity services as ordinary supporting infrastructure would contradict the threat model this design is built on.

### 1.2 Security levels

IEC 62443 defines security levels SL-1 through SL-4, describing the capability of an attacker the zone is expected to resist.

| Level | Resists |
|-------|---------|
| SL-1 | Casual or coincidental violation |
| SL-2 | Intentional violation using simple means, low resources, generic skills |
| SL-3 | Intentional violation using sophisticated means, moderate resources, industrial control system specific skills |
| SL-4 | Intentional violation using sophisticated means, extended resources, industrial control system specific skills |

SL-2 is the practical minimum for operational technology (OT) zones. SL-3 is assigned in this design where the consequence of compromise is production stoppage measured in shifts, physical risk, or cascading failure across other zones. No zone is assigned SL-4; that level implies resistance to a well-resourced state-level adversary and would not be a credible claim for a plant of this size, nor achievable with the controls available here.

**Target security level is an aspiration, not a claim of achievement.** This document records what each zone should resist. It does not assert that the implementation in Packet Tracer and VirtualBox achieves it.

---

## 2. Zone definitions

### Zone 1 — Safety

**Assets:** Safety instrumented system (SIS); fire detection; dust extraction monitoring; explosion suppression.
**Target:** Network-independent operation. No security level assigned.

The defining property of this zone is that it must perform its function correctly with the network entirely unavailable. It is listed here for completeness of the asset partition, not because it is a network zone in the normal sense.

Safety is not a security control and security is not a safety control. The SIS trips the boiler and the production lines when conditions become genuinely dangerous, and that behaviour must not depend on, or be influenceable by, anything in this design. Read-only alarm state may be presented to the supervisory zone for operator visibility. No path exists in the other direction. There is no remote access to this zone under any circumstances, including for vendors.

**Real implementation note:** in a real plant this is typically a separate physical network from a different vendor, with hardwired interlocks rather than network messaging. Nothing in Packet Tracer models this. The zone is documented and diagrammed but not simulated.

### Zone 2 — Cogeneration control

**Assets:** Boiler management system; turbine governor; remote terminal units (RTU) for fire water pumping, the electrical substation, and biomass silos.
**Target:** SL-3.

The highest-consequence zone on the site. Loss of control here causes the safety system to trip the boiler — correct behaviour, and not itself a disaster. The disaster is the recovery time. A biomass boiler restart is measured in hours, and while it is down there is no steam. Without steam the drying kilns cannot run, and because kiln drying consumes the majority of a sawmill's energy, the entire plant backs up behind the driers. A control network outage of minutes produces a production loss measured in shifts.

This zone also contains the interface to the national grid connection, which introduces an external commercial relationship and a regulatory dimension absent from the rest of the plant.

Vendor access is required periodically for turbine and boiler diagnostics and is brokered exclusively through the jump host in Zone 11. There is no direct external path.

### Zone 3 — Sawmill process control

**Assets:** Sawline PLCs; remanufacturing PLCs; human-machine interface (HMI) panels; field sensors and actuators.
**Target:** SL-3.

Real-time deterministic control of the primary breakdown line and the remanufacturing and moulding line. Traffic is industrial protocol traffic — PROFINET, EtherNet/IP, Modbus TCP — with strict latency requirements. Loss stops production immediately, though unlike Zone 2 recovery is fast once control is restored.

This zone accepts no inbound connections from any zone above it. Data leaves it; instructions do not enter it from outside except from Zone 4 within the defined conduit.

**Real implementation note:** the physical topology in a plant of this type is a redundant ring rather than a star, using a protocol such as Resilient Ethernet Protocol or Media Redundancy Protocol, so that a single cable damaged by a forklift does not isolate a machine centre. Packet Tracer does not model these protocols. Rapid Spanning Tree Protocol is configured instead, and the divergence is documented in the switching build.

### Zone 4 — Optimisation

**Assets:** Scanner optimiser consoles.
**Target:** SL-3.

Separated from Zone 3 for the reason set out in section 1.1: these are commodity computers with databases inside a control path, and they carry the patching and vulnerability profile of an IT asset while carrying the availability requirement of a control asset.

The commercial argument for treating this zone carefully is direct. Optimisation systems apply three-dimensional geometry to cutting decisions and materially improve lumber recovery over rule-based sawing. In a mill of this scale a single percentage point of recovery is worth a substantial sum annually. Degradation to default cutting patterns is not a minor inconvenience; it is continuous revenue loss that may not be immediately visible on any alarm screen.

Isolating these hosts in their own zone allows a patching and maintenance policy appropriate to Windows servers to be applied without granting that same policy to the PLCs in Zone 3.

### Zone 5 — Kiln control

**Assets:** Kiln controllers.
**Target:** SL-2.

Separated from Zone 3 because the protection requirement genuinely differs. Kilns are slow thermal processes with hours of inertia. Loss of supervisory control does not stop the process; the local controller continues executing its curve. What is lost is the drying record for the duration of the outage, which is a quality and traceability problem rather than an operational one.

That lower urgency is precisely what justifies a separate zone: it permits a maintenance window policy that would be unacceptable in Zone 3, and it prevents kiln systems inheriting controls sized for a different risk.

The zone has a functional dependency on Zone 2 — no steam, no drying — but that dependency is physical rather than logical, and creates no conduit.

### Zone 6 — Plant supervisory

**Assets:** Sawmill and utilities supervisory control and data acquisition (SCADA); data historian; operator workstations in cabins, the quality laboratory, the boiler control room and the yard dispatch office.
**Target:** SL-3.

The operational view of the plant. This zone reads from the control zones and writes nothing to them beyond defined setpoint changes originated by an authenticated operator. It holds the historian, which is the site's long-term record of process variables and therefore part of the audit evidence chain for certification.

Assigned SL-3 rather than SL-2 because a compromised supervisory host is a natural pivot point toward the control zones and because the historian's integrity underpins compliance evidence.

### Zone 7 — Physical security

**Assets:** Closed-circuit television (CCTV) cameras; network video recorders and video management servers; internal door access control panels and readers.
**Target:** SL-2.

Isolated for two independent reasons.

First, bandwidth. Sixty to one hundred and ten cameras produce the largest sustained traffic volume on the site. Allowing that traffic to share infrastructure with control traffic risks starving latency-sensitive industrial protocols.

Second, and more importantly, exposure. Network-attached cameras are among the most consistently vulnerable device classes in any industrial environment: default credentials, firmware that is rarely updated, and vendors who abandon support. They are numerous, physically accessible in a yard where contractors and truck drivers move, and impossible to individually monitor. The design assumption is that a camera will eventually be compromised. The zone exists so that when it happens, the attacker's position is worth as little as possible.

No inbound access from this zone to any other zone is permitted. Video is retrieved by pull from the video management server, not pushed outward.

### Zone 8 — Weighbridge and gatehouse

**Assets:** Weighbridge terminal and load cells; vehicle barriers; gate card and biometric readers; turnstiles.
**Target:** SL-2.

The zone most likely to be questioned by a reviewer, and the one with the strongest argument behind it.

The weighbridge is the entry point of the chain-of-custody record. Every inbound log truck and every outbound product load is weighed and identified here: supplier, species, volume, origin. That record is the basis for goods receipt, for supplier payment, for invoicing, and for the traceability claims that Forest Stewardship Council (FSC) chain of custody certification and the European Union Deforestation Regulation (EUDR) require. Its integrity is a commercial and a compliance matter, not merely an operational one.

It is also the most exposed location on the site. The gatehouse is a small unstaffed or lightly staffed structure at the perimeter, physically accessible to every truck driver who arrives, with network cabling running out to it across open ground. Placing it inside the enterprise zone would extend that zone to the fence line. Placing it inside a plant zone would give a perimeter structure a path into production.

A dedicated zone with a narrow conduit to the manufacturing execution system in Zone 9 and nothing else is the proportionate answer. It is also a shift-critical availability case in its own right: a weighbridge down at shift change queues trucks at the gate, and the queue does not clear quickly.

### Zone 9 — Plant operations

**Assets:** Manufacturing execution system (MES); laboratory information management system (LIMS); computerised maintenance management system (CMMS); label printers; handheld scanners and forklift-mounted terminals; yard and warehouse wireless infrastructure.
**Target:** SL-2.

The manufacturing operations management layer, corresponding broadly to Purdue Level 3. It coordinates production against targets, tracks batch genealogy from log intake to finished pack, and manages quality and maintenance.

Two members of this zone deserve specific mention. **Label printers** are production-critical despite appearing mundane: a pack without a label cannot ship, so a printer failure stops dispatch as effectively as a line fault. **Yard wireless** is the only wireless infrastructure permitted to carry operational traffic, and it is a separate service set from any corporate or guest wireless, mapped to this zone alone.

### Zone 10 — Site infrastructure

**Assets:** Site domain controllers; Domain Name System (DNS); Dynamic Host Configuration Protocol (DHCP); network time source; backup infrastructure; core switching and network management.
**Target:** SL-3.

The zone that this design treats as the primary asset to defend, for the reasons set out in section 1.1. Authentication failure cascades into every dependent system; compromise of a domain controller is compromise of everything that trusts it.

Three specific design positions follow from that, and each is implemented later in the project:

**Backup credential isolation.** Backup infrastructure resides in this zone but does not authenticate against the domain it protects. Attackers target backup systems deliberately and early, and a backup account with domain administrative rights means the domain and its recovery mechanism fall together. This is the most common single failure in real ransomware recoveries.

**Time as a security asset.** The site time source is the reference for production records, controller events, security logs and audit evidence. Time drift does not fail loudly; correlation degrades quietly until an investigation or an audit reveals that records cannot be reconciled. The stratum design is documented in the routing and services build.

**Administrative tiering.** Accounts with administrative rights in this zone are not used to log on to workstations in Zone 12. Credential reuse across tiers is what converts a single compromised workstation into a domain compromise.

### Zone 11 — Industrial demilitarised zone (IDMZ)

**Assets:** Historian replica; patch relay; vendor jump host; reverse proxy.
**Target:** SL-3.

The brokered boundary between the enterprise and the plant. The controlling rule is stated once and enforced everywhere: **no protocol traverses the IT/OT boundary directly in either direction.** Every flow terminates in this zone and is re-originated.

The zone provides three services:

**Data out.** The historian in Zone 6 replicates to a replica here. Management reporting in Zone 12 reads the replica. No enterprise host ever connects to the production historian.

**Patches in.** Operating system updates are received here, staged, tested against a maintenance schedule, and then pulled by OT hosts. OT systems never reach the internet, and patch distribution never originates from the enterprise zone.

**Vendors in.** Equipment vendors — turbine, moulder, optimiser, kiln suppliers — require periodic diagnostic access. They connect to a jump host here, authenticate with multi-factor authentication, and their sessions are recorded. They never obtain a route into a plant zone. Access is enabled on request and disabled afterwards rather than standing open.

**Real implementation note:** a production IDMZ of this kind is normally built with a firewall pair from different vendors, so that a single vulnerability does not open both boundaries. This design uses a single firewall, which is a deliberate compromise for the tooling available and is recorded as such.

### Zone 12 — Enterprise

**Assets:** ERP; logistics, customs and export documentation systems; forest management and geographic information system (GIS); business intelligence; corporate workstations and multifunction printers; internet protocol telephony.
**Target:** SL-2.

Ordinary business systems: finance, purchasing, payroll, sales, export documentation. It holds the systems that carry EUDR due diligence statements and port and customs bookings, which is why its availability requirement is higher than a general office network would suggest — a shipment held at a border is expensive.

This zone has internet access and hosts the encrypted tunnel to the Brazilian parent company. It is the zone where phishing lands and where a compromise most probably begins. The design assumes that.

### Untrusted segment — guest and contractor

**Assets:** Guest wireless; contractor devices.
**Target:** No security level. Explicitly outside the trust boundary.

Not a zone in the IEC 62443 sense, and named separately to make that clear. Traffic is routed directly to the internet and has no path to any zone. It is documented here because an undocumented guest network is a common finding in real assessments, and because construction and commissioning contractors will be present on this site for an extended period.

---

## 3. Conduits

A conduit is a permitted communication path between zones, with defined protocols, direction and controls. **Any flow not listed here is denied.** The default posture is deny-all; the table below is the complete set of exceptions.

Direction is expressed as initiator to responder. A conduit permitting A to B does not permit B to initiate to A.

| ID | From | To | Purpose | Protocols | Controls |
|----|------|-----|---------|-----------|----------|
| C-01 | Z1 Safety | Z6 Supervisory | Alarm and trip state for operator visibility | Hardwired contact or read-only serial | Physically unidirectional. No reverse path exists |
| C-02 | Z2 Cogeneration | Z6 Supervisory | Boiler and turbine process data | OPC Unified Architecture, read-only | Stateful inspection; specific host pairs only |
| C-03 | Z3 Sawmill control | Z6 Supervisory | Machine state, counts, fault codes | OPC UA, read-only | Stateful inspection; specific host pairs only |
| C-04 | Z4 Optimisation | Z3 Sawmill control | Cutting solutions to sawline controllers | Vendor protocol over TCP | Latency-critical. Restricted to named host pairs |
| C-05 | Z4 Optimisation | Z6 Supervisory | Per-log geometry, yield statistics | SQL, read-only | Specific database endpoints only |
| C-06 | Z5 Kiln control | Z6 Supervisory | Drying curves and charge records | OPC UA, read-only | Stateful inspection |
| C-07 | Z6 Supervisory | Z2, Z3, Z5 | Operator setpoint changes | OPC UA write, named tags only | Authenticated operator session. Restricted tag set. Logged and retained |
| C-08 | Z6 Supervisory | Z11 IDMZ | Historian replication to replica | Vendor replication protocol | One-way push. Initiated from Z6. Z11 never initiates toward Z6 |
| C-09 | Z8 Weighbridge | Z9 Plant operations | Weight tickets, load identification | HTTPS to MES endpoint | Single destination. Client certificate authentication |
| C-10 | Z9 Plant operations | Z6 Supervisory | Production orders and schedule | HTTPS | Application-layer authentication |
| C-11 | Z9 Plant operations | Z11 IDMZ | Production data for enterprise consumption | HTTPS to reverse proxy | Terminated and re-originated at the proxy |
| C-12 | Z2, Z3, Z4, Z5, Z6, Z7, Z8, Z9 | Z10 Site infrastructure | Authentication, name resolution, address assignment, time | LDAP, Kerberos, DNS, DHCP relay, NTP | Restricted to domain controllers and the time source. No general access to the zone |
| C-13 | Z10 Site infrastructure | All OT zones | Backup collection | Backup agent protocol | Scheduled windows. Backup service account is not a domain account |
| C-14 | Z11 IDMZ | Z2, Z3, Z4, Z5, Z6, Z9 | Patch distribution | HTTPS, pull only | OT hosts initiate. The patch relay never initiates toward OT |
| C-15 | Z11 IDMZ | Z2, Z3, Z4, Z5 | Vendor diagnostic sessions | RDP or SSH from jump host | Multi-factor authentication. Session recording. Enabled on request, disabled after |
| C-16 | Z12 Enterprise | Z11 IDMZ | Management reporting against the historian replica | HTTPS to reverse proxy | Terminates at the proxy. No path beyond it |
| C-17 | Z12 Enterprise | Z10 Site infrastructure | Enterprise authentication and name resolution | LDAP, Kerberos, DNS | Restricted to domain controllers. Administrative tier separation enforced |
| C-18 | Z12 Enterprise | Internet | Business systems, email, web, customs and port systems | HTTPS, SMTP | Perimeter firewall. Web and mail filtering |
| C-19 | Z12 Enterprise | Brazilian parent | Corporate systems and consolidated reporting | IPsec site-to-site tunnel | Encrypted tunnel. Defined subnets only |
| C-20 | Z7 Physical security | Z10 Site infrastructure | Time synchronisation only | NTP | Sole outbound flow permitted from this zone |
| C-21 | Z12 Enterprise | Z7 Physical security | Video review by authorised security staff | HTTPS to video management server | Pull only. Named user group. Cameras themselves unreachable from Z12 |

### 3.1 Flows explicitly denied

Stating what is forbidden is as much a part of the design as stating what is permitted. The following are denied by policy and the denial is verified by testing in Phase 4:

- Any direct connection between Zone 12 (Enterprise) and any OT zone. All such traffic terminates in Zone 11.
- Any inbound connection from the internet to any zone other than published services in the enterprise demilitarised zone.
- Any connection originating in Zone 7 (Physical security) toward any zone other than C-20.
- Any connection into Zone 1 (Safety) from anywhere.
- Any vendor access that does not traverse the Zone 11 jump host.
- Any use of Zone 10 administrative credentials on Zone 12 workstations.
- Any path between the untrusted guest segment and any zone.

---

## 4. Consequences for the addressing and VLAN design

Three requirements follow directly from this document and constrain the next design step:

**Zones map to distinct address ranges.** A zone whose addresses are scattered across the site's address space cannot be expressed as a small number of access control list entries. Each zone receives a contiguous range, summarisable in a single statement.

**Growth is provisioned within zones, not appended after them.** The plant opens at approximately 150 employees and is designed for 400. Each zone's range is sized against the 400 figure at the outset. Renumbering a production network is disruptive to the point of being avoided even when it is clearly needed, which means the plant would run permanently with a compromised design.

**The OT/IT boundary is a routing boundary, not only a filtering boundary.** Enterprise address space and plant address space are drawn from clearly distinguishable ranges so that a misconfiguration is visible on inspection rather than silent.

---

## 5. Acronyms

CCTV, closed-circuit television. CMMS, computerised maintenance management system. DHCP, Dynamic Host Configuration Protocol. DNS, Domain Name System. ERP, enterprise resource planning. EUDR, European Union Deforestation Regulation. FSC, Forest Stewardship Council. GIS, geographic information system. HMI, human-machine interface. IDMZ, industrial demilitarised zone. IT, information technology. LDAP, Lightweight Directory Access Protocol. LIMS, laboratory information management system. MES, manufacturing execution system. NTP, Network Time Protocol. OPC UA, OPC Unified Architecture. OT, operational technology. PERA, Purdue Enterprise Reference Architecture. PLC, programmable logic controller. RDP, Remote Desktop Protocol. RTU, remote terminal unit. SCADA, supervisory control and data acquisition. SIS, safety instrumented system. SL, security level. SSH, Secure Shell. VLAN, virtual local area network.
