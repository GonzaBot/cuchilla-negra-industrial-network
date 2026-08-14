# Cuchilla Negra Industrial Network

Greenfield network and systems infrastructure design for a fictional pine sawmill with biomass cogeneration and a remanufacturing line, located in Rivera, Uruguay.

---

## Disclaimer

Aserradero Cuchilla Negra S.A. does not exist. It is a fictional company modelled on publicly available information about the Uruguayan and Brazilian forestry-timber sector and on the general characteristics of sawmill operations. No proprietary, internal or confidential information about any real company is used or implied anywhere in this repository. All addresses, hostnames, credentials and topologies are invented for this exercise.

---

## The plant

A 50-hectare greenfield site producing sawn and remanufactured pine for export. The plant runs a log yard and weighbridge, debarking and optical scanning, a primary breakdown sawline with cut optimisation, drying kilns, a planer and moulding line, and a biomass cogeneration plant fuelled by mill residue. It opens with approximately 150 employees and is designed to grow to 400 without renumbering the network.

Kiln drying consumes the majority of a sawmill's energy, which is why the cogeneration plant burns the mill's own residue rather than existing as a separate business. The cogeneration control system is the highest-consequence system on site.

---

## Design approach

The network is segmented using the Purdue Enterprise Reference Architecture (PERA) as shared vocabulary and IEC 62443 zones and conduits as the actual design method. Zones are defined by risk and protection requirement rather than by device type, which means a zone may sit inside one Purdue level or span several. Every conduit between zones is documented with its protocols, direction, ports and controls.

An industrial demilitarised zone (IDMZ) sits between the business network and the production network. No system communicates directly across the information technology (IT) and operational technology (OT) boundary in either direction.

---

## Threat model

Recent industrial incident data shows that ransomware operators rarely manipulate control systems directly. Production stops because the enterprise systems that production depends on — identity services, virtualisation, enterprise resource planning (ERP), remote access gateways — are encrypted or shut down precautionarily.

This design therefore treats Active Directory Domain Services (AD DS) and the virtualisation layer as the primary assets to defend, not as supporting infrastructure.

---

## Compliance drivers

Forest Stewardship Council (FSC) chain of custody requires records to be retained for five years. The European Union Deforestation Regulation (EUDR) requires due diligence documentation to be retained for five years and to be retrievable within 24 hours of a competent authority request.

These requirements set the recovery time and retention objectives used in the backup and disaster recovery design, rather than those objectives being invented for the exercise.

---

## Tools, and what each is responsible for

| Tool | Responsibility |
|------|----------------|
| Cisco Packet Tracer | Routing, switching, firewall, wireless, access control lists, end devices |
| VirtualBox with Windows Server 2025 | Active Directory, DNS, DHCP, Group Policy, file services, backup and restore |
| Visual Studio Code | Documentation and configuration authoring |
| Git and GitHub | Version control and portfolio delivery |

Servers built in VirtualBox correspond to specific devices on the Packet Tracer topology. That correspondence is documented explicitly in `docs/03-servers/`.

---

## Simulated versus implemented

Packet Tracer is a simulator. Its generic server models Dynamic Host Configuration Protocol (DHCP), Domain Name System (DNS), Hypertext Transfer Protocol (HTTP), Trivial File Transfer Protocol (TFTP), authentication, authorisation and accounting (AAA), email and File Transfer Protocol (FTP) only. There is no Windows, no Lightweight Directory Access Protocol (LDAP), and no Group Policy.

It also cannot model industrial protocols such as PROFINET, Modbus TCP or OPC Unified Architecture (OPC UA). Operational technology traffic is represented by endpoints and enforced by access control lists rather than genuinely simulated.

Where the simulator cannot do something the design requires, the documentation states what the real implementation would be and marks it as designed but not implemented. Nothing in this repository claims to be running that is not running.

---

## Repository structure

```text
docs/01-design/       Industry context, asset inventory, zones and conduits,
                      addressing and VLAN plans
docs/02-network/      Switching, routing, security and wireless build
                      documentation
docs/03-servers/      Windows Server and Active Directory documentation
docs/04-operations/   Monitoring, backup, disaster recovery, runbooks
configs/              Device configurations as plain text
diagrams/             Logical and physical topology
evidence/             Verification screenshots
packet-tracer/        Packet Tracer topology files
scripts/              PowerShell and shell scripts
```

---

## Status

| Phase | Scope | Status |
|-------|-------|--------|
| 0 | Repository foundations | Complete |
| 1 | Design: zones, addressing, VLANs, topology | In progress |
| 2 | Switching | Not started |
| 3 | Routing, WAN, time synchronisation | Not started |
| 4 | Access control, firewall, IDMZ, wireless | Not started |
| 5 | Active Directory and services | Not started |
| 6 | Backup, recovery and continuity | Not started |
| 7 | Documentation review | Not started |

---

## Glossary

The full glossary is maintained in `docs/GLOSSARY.md`. Acronyms are expanded on first use in every document in this repository.

---

## Author

Gonzalo — systems analyst, Rivera, Uruguay.