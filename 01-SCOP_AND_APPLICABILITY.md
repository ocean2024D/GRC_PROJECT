# Step 1 — Concepts & Scope

**Project:** NVIDIA-REG-RD-2026-1106 — Regional R&D and Production Hub
**Dossier audited:** Netify Architecture Group submission (design proposal, network analysis document, cost estimation, Packet Tracer file and device configs)
**Date:** [23/09/2026]
**Auditors:** [Derya, Kien, Lakshmi]

---

## 1. Asset Inventory

*Sectors, VLANs, devices, services and the data each one holds. Built from the RFQ, the design documents (`NVIDIA_Technical_Commercial_Proposal_Final.pdf`, `Network_Analysis.pdf`, `cost_estimation_.xlsx`) and the Packet Tracer configs (`Netify_Final.pkt` and exported device configs).*

| # | Sector / Zone | VLAN | Subnet | Devices | Services running | Data held |
|---|---|---|---|---|---|---|
| 1 | Support Sector A | VLAN 10 | 192.168.10.0/24 | Switch-Support-A (Catalyst 2960), 10 workstations, 1 network printer (.17) | DHCP client, DNS client, port security (sticky MAC learning, violation = restrict) | End-user work files; personal data of Support A staff (accounts, print jobs) |
| 2 | Support Sector B | VLAN 20 | 192.168.20.0/24 | Switch-Support-B (Catalyst 2960), 10 workstations, printer | DHCP client, DNS client | End-user work files; personal data of Support B staff |
| 3 | Production Sector | VLAN 30 | 192.168.30.0/24 | Switch-Production (Catalyst 2960), 10 Dell Precision high-end AI towers | DHCP client, DNS client | Proprietary R&D data, LLM training data/models, software source code — the most sensitive data on the site per the RFQ's stated purpose |
| 4 | Management / Secretariat | VLAN 40 | 192.168.40.0/24 | Switch-Management (Catalyst 2960), 5 workstations, printer | DHCP client, DNS client | Administrative/business records; likely includes personal data of staff (HR-adjacent) |
| 5 | IT Department | VLAN 50 | 192.168.50.0/24 | Switch-IT (Catalyst 2960), 5 workstations | DHCP client, DNS client, administrative access to AAA server and network devices | Network device credentials, admin account data, configuration backups |
| 6 | Study Sector | VLAN 60 | 192.168.60.0/24 | Switch-Study (Catalyst 2960), 8 workstations | DHCP client, DNS client | Training/reference material, internal documentation |
| 7 | Server farm | VLAN 100 | 192.168.100.0/24 | Core-Switch (Catalyst 3560, multilayer); AAA/RADIUS server (.4); DNS server / DHCP-helper target (.3); Web/FTP "Application" server (.2 — referenced by the original per-sector ACLs) | DNS resolution, DHCP relay (`ip helper-address`), AAA authentication + accounting (RADIUS), inter-VLAN routing, ACL enforcement | RADIUS shared secret and credential hashes (`Cisco123`, `vpnuser` hash), DNS records, FTP-stored files, web content |
| 8 | Application / "DMZ" | VLAN 200 | 192.168.200.0/24 | "Application Server" — named **"Application Hub Platform"** in the design proposal §3, described as hosting HTTP+FTP in `Network_Analysis.pdf`, and labelled **"FTP Server"** in the cost spreadsheet's Network Parameters tab | HTTP (custom landing page), FTP (tested with `cisco`/`cisco` credentials per `Network_Analysis.pdf`) | Public-facing web content, FTP-stored files. **Note:** the VLAN and its SVI exist on the core switch, but none of the ACLs in the supplied configs reference 192.168.200.0/24 as a source or destination — flagged for verification in Step 3 |
| 9 | Perimeter | — (outside/inside) | 10.0.8.0/30 (outside), 10.0.8.4/30 (inside) | Cisco ASA 5506-X edge firewall | Stateful inspection, static routing to ISP, SSL clientless VPN (`webvpn`), inspection of DNS/FTP/ICMP/TFTP | VPN user credential (`vpnuser`), shared secrets, full inbound/outbound traffic control |
| 10 | Management plane (cross-cutting) | — | — | All 6 access switches, core switch, firewall | Console and VTY (Telnet/SSH) access; AAA login integration only confirmed on the core switch | Device running-configs, local/enable passwords, administrative session data |

**Assets described in the design documentation but not evidenced in the supplied Packet Tracer configs** (documented ≠ implemented — to be tested in Step 3, not asserted here):
- Cisco ISR 4331 edge/ISP services router (listed in BOM and topology narrative; no separate config file supplied)
- Cable modem (listed in `Network_Analysis.pdf` component list; no config or IP evidence supplied)
- VTY/console passwords or AAA references on the five non-core switches (`login` is present with no password or `aaa` line visible in the configs supplied)

---

## 2. Applicability Note

*Per the decision flow in `02_Regulations.md` §2.5. Only regulations marked **Applies** may be cited as the basis for a finding in later steps.*

**Entity Context:** The hub is operated by a Belgian legal entity registered in the manufacturing sector (a NIS2 sector). The entity directly employs 48 staff with an annual turnover of €7M. Because it is a wholly owned subsidiary of NVIDIA Corporation, corporate group aggregation rules apply under NIS2, incorporating the consolidated headcount and revenue of the entire corporate group.

| Regulation / standard | Applies? | Reasoning |
|---|---|---|
| **GDPR** | **Applies** | The hub employs staff across six sectors (48 workstations); their identities, credentials and access logs are personal data processed by a Belgian legal entity. |
| **NIS2 / Belgian NIS2 Act** | **Applies** *(Important Entity)* | The facility operates in manufacturing (a regulated NIS2 Annex sector) in Belgium. While the local entity independently falls below the enterprise threshold (48 staff, €7M turnover), its status as a wholly owned subsidiary of NVIDIA Corporation requires consolidated group aggregation. The parent group's total size places the entity in scope as an **Important Entity**. |
| **CyFun (CCB)** | **Applies** | A direct consequence of NIS2 applying: CyFun is the CCB's default practical framework for demonstrating NIS2 compliance in Belgium, and is used throughout this audit as a source of controls regardless of which of the three compliance routes (CyFun / ISO 27001 / inspection) the entity ultimately pursues. |
| **ISO/IEC 27001:2022** | **Uncertain** | No document in the dossier states whether NVIDIA or Netify has chosen ISO 27001 certification as its NIS2 compliance route. Would need to know which of the three CCB-recognised routes (CyFun, ISO 27001, or CCB inspection) applies. Annex A is still used in this audit as a control source independent of this answer — see `03_AuditMethod.md`. |
| **Cyber Resilience Act (CRA)** | **Does not apply** | CRA governs products with digital elements placed on the EU market. This RFQ is for NVIDIA's internal R&D/production network — infrastructure NVIDIA uses, not a product NVIDIA sells. NVIDIA's actual hardware/software products may separately be in CRA scope, but that is outside this design's boundary. |
| **AI Act** | **Applies** *(Risk tier uncertain)* | The RFQ explicitly requires workstations for "LLM training" and "AI creation" (§2) — AI systems will be developed on this network, which is enough to bring the AI Act into scope. The specific risk classification (general-purpose model, limited-risk, high-risk) cannot be determined from the RFQ or design documents, which say nothing about the AI systems' intended deployment or use case. |
| **DORA** | **Does not apply** | NVIDIA's regional R&D/Production Hub is not a bank, insurer, investment firm, payment institution or other financial entity, and nothing in the RFQ suggests financial-sector activity. |
| **SOC 2** | **Does not apply** | SOC 2 is a report a SaaS/technology provider furnishes to its enterprise customers. This hub is internal R&D/production infrastructure, not a service NVIDIA sells to external clients. |

---

## Notes carried forward to Step 2

- Only **GDPR**, **NIS2/Belgian NIS2 Act** (as an Important Entity), and **CyFun** may be cited as the regulatory basis for findings without further caveat. **AI Act** findings are possible but should flag the unresolved risk-tier question. **ISO 27001** is used as a control source (Annex A) regardless of the "applies" uncertainty, per the method document.
- The VLAN 200 naming inconsistency (Application Hub Platform / DMZ / FTP Server across three documents) and the apparent absence of any ACL governing VLAN 200 traffic are carried forward as candidate check subjects for Step 2, not asserted as findings here.
- The five non-core switches show `login` on VTY lines with no visible password or AAA reference — carried forward as a candidate access-control check.
