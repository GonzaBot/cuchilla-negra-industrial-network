# Asset Inventory

**Document:** `docs/01-design/01-asset-inventory.md`
**Phase:** 1 — Design
**Status:** Baseline

---

## Purpose

IEC 62443 is a risk-driven standard. Zones cannot be defined until the assets are known and the consequence of losing each one is understood. This document is therefore the first design artefact produced for Aserradero Cuchilla Negra S.A., and everything that follows — zone definitions, conduits, addressing, virtual local area network (VLAN) allocation, access control lists — derives from it.

The Purdue Enterprise Reference Architecture (PERA) level is recorded for each asset class as shared vocabulary. It is not the design method. Zones are defined in `02-zones-and-conduits.md` by protection requirement, which produces zones that sit inside a single Purdue level, zones that span several, and several zones within one level.

---

## Scope boundaries

**In scope.** Everything inside the 50-hectare site perimeter: the log yard, weighbridge and gatehouse, the sawmill building, drying kilns, the remanufacturing and moulding line, the cogeneration plant, the warehouse and dispatch area, and the administration building.

**Out of scope.** Forest harvesting equipment. Harvesters and forwarders operate off-site, usually under contract to third parties, on land that is not the plant. They are not assets on this network.

The *data* they produce is in scope. Harvest volumes and plot-level geolocation coordinates — the latter being mandatory evidence under the European Union Deforestation Regulation (EUDR) — arrive from contractors as files or application programming interface (API) calls over the internet, and are received by the Level 4 geographic information system (GIS). That external data flow is secured at the perimeter and documented as a conduit.

**Also out of scope.** Pellet production. The plant produces sawn timber and remanufactured mouldings; residues feed the cogeneration boiler rather than a pellet line.

---

## Sizing basis

Two counts are given for each asset class: at opening (approximately 150 employees) and at full development (approximately 400 employees). The addressing plan is sized against the 400 figure so that growth does not require renumbering.

---

## Availability classes

| Class | Meaning |
|-------|---------|
| Continuous | Required 24 hours a day while the plant runs |
| Shift-critical | Failure is tolerable briefly but stops work at shift boundaries or during specific operations |
| Business hours | Failure is an inconvenience deferred to the next working day |
| Network-independent | Must function correctly with the network completely unavailable |

---

## Inventory

| # | Asset class | PERA | Function | Data produced | Consequence of a five-minute outage | Availability | 150 / 400 |
|---|-------------|------|----------|---------------|--------------------------------------|--------------|-----------|
| 1 | Field sensors and actuators | 0 | Moisture, position, temperature and pressure sensing; motors, variable frequency drives (VFD), pneumatic and hydraulic cylinders | Analogue and digital signals to controllers | Local machine stops | Continuous | ~2000 / ~4000 |
| 2 | Safety instrumented system (SIS) | 0–1 | Trips the boiler and production lines on genuinely dangerous conditions | Trip and demand events | None, provided it is independent — that independence is its defining property | Network-independent | 3 / 5 |
| 3 | Fire detection, dust extraction and explosion suppression | 0–1 | Life safety in a combustible wood-dust environment | Alarm states to a monitored panel | Loss of visibility only; the system itself continues to function | Network-independent | 1 system |
| 4 | Sawline programmable logic controllers (PLC) | 1 | Feed speed and saw synchronisation on the primary breakdown line | Machine state, piece counts, fault codes | Sawline stops | Continuous | 8 / 14 |
| 5 | Remanufacturing PLCs | 1 | Moulder, optimising crosscut and finger-jointing control | Piece-level reject and yield data | Remanufacturing line stops | Continuous | 5 / 12 |
| 6 | Kiln controllers | 1 | Executes drying curves for each charge | Temperature, humidity and schedule per charge | Nothing immediate — thermal inertia is measured in hours. A gap appears in the drying record, which is a quality and traceability problem rather than an operational one | Continuous | 6 / 12 |
| 7 | Boiler management system and turbine governor | 1 | Flame safety and turbine speed regulation for cogeneration | Combustion, steam and generation data | The safety system trips the boiler, which is correct behaviour. Restart takes hours. Without steam the kilns starve, and the whole plant backs up behind the driers | Continuous — highest consequence on site | 2 / 2 |
| 8 | Remote terminal units (RTU) — fire water pumping, electrical substation, biomass silos | 1 | Remote field control and monitoring | Status, flow and level telemetry | Loss of remote visibility; local control continues | Continuous | 4 / 8 |
| 9 | Access control panels, card and biometric readers, turnstiles, vehicle barriers | 1 | Physical access control at the gate and internal doors | Entry and exit events | Gate falls back to manual operation; trucks queue | Shift-critical | 12 / 25 |
| 10 | Weighbridge terminal and load cells | 1 hardware, 3 data | Weighs every inbound log truck and every outbound product load | Supplier, species, volume and origin — the first record in the chain of custody | Trucks queue at the gate. No weight means no goods receipt and no invoice. The consequence is commercial, not just operational | Shift-critical | 2 / 3 |
| 11 | Human-machine interface (HMI) panels | 2 | Operator interface at the machine | Operator actions and alarm acknowledgements | Operators lose view; the line continues running blind | Continuous | 15 / 30 |
| 12 | Sawmill and utilities supervisory control and data acquisition (SCADA) | 2 | Plant-wide supervisory view | Production flow, alarms, energy consumption, volumetric yield | Loss of view across production | Continuous | 2 / 3 |
| 13 | Scanner optimiser consoles | 2, with Level 1 consequences | Acquires three-dimensional log geometry and computes the optimal cutting pattern | Per-log geometry, cutting solutions, recovery statistics | The sawline stops or falls back to default patterns. Because optimisation drives lumber recovery directly, this is immediate revenue loss | Continuous | 4 / 7 |
| 14 | Network video recorders and video management servers | 2 | Closed-circuit television (CCTV) recording and retention | Video streams — the largest single bandwidth consumer on site | A gap in recorded footage; no production impact | Continuous | 2 / 4 |
| 15 | CCTV cameras | 2 | Yard, production lines, boiler house and perimeter surveillance | Video | Individual blind spots | Continuous | 60 / 110 |
| 16 | Operator workstations | 2 | Sawmill cabins, quality laboratory, boiler control room, yard dispatch office | Local operator input | A single position degraded | Shift-critical | 20 / 40 |
| 17 | Operational technology (OT) access switches, industrial DIN-rail | 2–3 | Connect PLCs, HMIs and cameras within the plant | Network telemetry | Everything behind the switch is isolated | Continuous | 25 / 45 |
| 18 | Manufacturing execution system (MES) | 3 | Tracks batches from log intake through to finished packs | Batch genealogy and chain-of-custody linkage | Traceability records stop accumulating | Shift-critical | 1 / 1 |
| 19 | Data historian | 3 | Long-term storage of process variables | Kiln curves, boiler pressures, downtime records | A gap in the historical record, which is also a gap in audit evidence | Continuous | 1 / 1 |
| 20 | Laboratory information management system (LIMS) | 3 | Finger-joint strength, moisture content and density testing | Test results tied to production batches | Results queue | Business hours | 1 / 1 |
| 21 | Computerised maintenance management system (CMMS) | 3 | Saw hours, preventive maintenance scheduling | Work orders and asset history | Deferred | Business hours | 1 / 1 |
| 22 | Label printers | 3 | Prints pack labels and barcodes at the end of the line | Pack identity | Dispatch and packing stop. A pack without a label cannot ship | Shift-critical | 8 / 15 |
| 23 | Handheld scanners and forklift-mounted terminals | 3 | Records pack movement in the yard and warehouse | Location and movement events | Movements are recorded on paper and reconciled later, with predictable error | Shift-critical | 20 / 45 |
| 24 | Yard and warehouse wireless — controller and access points | 3 | Coverage for mobile terminals across open yard areas | — | Handhelds and forklift terminals lose connectivity | Shift-critical | 18 / 35 |
| 25 | Plant core switches | 3 | Site network backbone | Network telemetry | Large-scale isolation | Continuous | 2 / 2 |
| 26 | Site domain controllers, Domain Name System (DNS), Dynamic Host Configuration Protocol (DHCP) | 3 | Identity, authentication and name resolution for the site | Authentication and audit logs | Logons fail, name resolution fails, and every dependent system cascades. Industrial incident data shows this is the layer attackers actually reach, and production stops as a consequence rather than through direct control system manipulation | Continuous — the primary asset to defend | 2 / 3 |
| 27 | Network time source | 3 | Common clock for logs, production records and controllers | — | Drift begins. Correlation between production records, controller events and security logs degrades silently rather than failing visibly, which makes this a slow-motion loss of audit evidence | Continuous | 2 / 2 |
| 28 | Backup infrastructure | 3 | PLC configurations, historian, MES and file data | Backup catalogue | No immediate impact, and catastrophic impact at the moment it is needed. Attackers target backup infrastructure deliberately, so its credentials must be isolated from the domain it protects | Continuous | 1 / 2 |
| 29 | Internet protocol (IP) telephony | 3–4 | Production offices and maintenance workshop | Call detail records | Radios and mobile telephones serve as fallback | Business hours | 40 / 90 |
| 30 | Industrial demilitarised zone (IDMZ) servers — historian replica, patch relay, vendor jump host | 3.5 | Brokered data exchange across the IT/OT boundary and controlled vendor access | Replicated production data, patch status, session recordings | Management reporting stops and vendor access is blocked. Production is unaffected, which is exactly the intent of the design | Business hours | 3 / 4 |
| 31 | Enterprise resource planning (ERP) | 4 | Finance, purchasing, payroll, sales | Commercial transactions | Invoicing and dispatch paperwork stop | Business hours | 1 / 1 |
| 32 | Logistics, customs and export documentation | 4 | Port bookings, customs clearance, EUDR due diligence statements | Shipment and compliance records | Shipments are delayed at the border | Business hours | 1 / 1 |
| 33 | Forest management and GIS | 4 | Plot geolocation and harvest projections; holds EUDR evidence | Geolocation data received from contractors | Deferred | Business hours | 1 / 1 |
| 34 | Business intelligence | 4 | Margin, yield and energy dashboards | Reports | Deferred | Business hours | 1 / 1 |
| 35 | Corporate workstations and multifunction printers | 4 | Administration, commercial and export, management | Office documents | Individual users affected | Business hours | 45 / 130 |
| 36 | Perimeter firewall and wide area network (WAN) routers | Boundary | Policy enforcement between all zones; encrypted tunnel to the Brazilian parent company | Session and policy logs | The site loses internet access and connectivity to the parent company | Continuous | 2 / 2 |

---

## Assets that resist clean classification

Three entries above are deliberately awkward, and the awkwardness is the design problem rather than a defect in the table.

**The scanner optimiser (13).** It performs two jobs simultaneously. It acquires sensor data and computes a cutting decision that it hands to the sawline controller within milliseconds, which is Level 1 behaviour: deterministic, real-time, inside the control loop. It is also a general-purpose industrial computer running a commodity operating system and a relational database, retaining per-log geometry and yield history for reporting, which is Level 2 or Level 3 behaviour. The result is a patchable, network-attached Windows host sitting inside a real-time control path — precisely the profile of a machine that gets encrypted in a ransomware event and stops production.

**The weighbridge (10).** Its hardware is Level 1: load cells and a controller. Its data is Level 3 and arguably Level 4, because it is the first record in the chain of custody and the basis for goods receipt and invoicing. It is physically located at the site perimeter, outside the plant buildings, which is a weak point in every sense. It therefore cannot be assigned to either the process control zone or the enterprise zone without creating a problem.

**Site identity services (26).** Formally a Level 3 asset. In practice it is the asset whose compromise stops the plant, because authentication failure cascades into every system that depends on it. Treating it as ordinary supporting infrastructure would contradict the threat model this design is built on.

These three cases are the reason zones are defined by protection requirement rather than by Purdue level.

---

## Glossary of acronyms used in this document

API, CCTV, CMMS, DHCP, DNS, ERP, EUDR, GIS, HMI, IDMZ, IP, LIMS, MES, OT, PERA, PLC, RTU, SCADA, SIS, VFD, VLAN, WAN. Each is expanded on first use above and consolidated in `docs/GLOSSARY.md`.
