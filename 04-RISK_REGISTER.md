# RISK REGISTER

## NVIDIA Regional R&D & Production Hub

**Project:** NVIDIA-REG-RD-2026-1106  
**Frameworks:** CyFun 2025 / NIST CSF 2.0, GDPR, Belgian NIS2 Act (Manufacturing Sector, Important Entity)

---

## Rating Scale & Methodology

*(Agreed before rating; applied uniformly)*  
$$\text{Risk} = \text{Likelihood} \times \text{Impact}$$ (each rated on a scale of 1 to 3).

| Likelihood | Description | Impact | Description |
| :--- | :--- | :--- | :--- |
| **3 - High** | No control in the way; common technique | **3 - High** | Sensitive data, whole sector, or production |
| **2 - Med** | Partial control, or attacker needs a foothold | **2 - Med** | One sector or one service |
| **1 - Low** | Several conditions align, or insider access | **1 - Low** | Limited, easily recovered |

* **High:** 6
* **Critical:** 3
* **Medium:** 4
* **Low:** 2

> **Note on Impact:** Structured as a pyramid rather than a flat wall of Critical ratings. Impact is anchored directly to the asset inventory: Production (VLAN 30), Server DMZ (VLAN 100), and IT (VLAN 50) weigh heaviest.

---

## Master Risk Register

*All 15 findings, ordered by priority.*

| # | ID | Finding | Check | Area | L | I | Score | Rating | Regulation (via control) | Cost |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | F-01 | Overly permissive inbound edge firewall policy (`ALLOW_ALL`) | SEG-02 | Segmentation | 3 | 3 | 9 | Critical | NIS2 21(2)(a) / ISO A.8.20/8.22 / CyFun PR.IR-01 | Zero (config) |
| 2 | F-02 | DMZ server farm terminated on core-switch SVI, not the firewall | SEG-03 | Segmentation | 3 | 3 | 9 | Critical | NIS2 21(2)(a) / ISO A.8.22 / CyFun PR.IR-01 | Zero (re-cable) |
| 3 | F-03 | Inter-VLAN isolation ACLs defined but not bound to SVIs | SEG-01 | Segmentation | 3 | 3 | 9 | Critical | NIS2 21(2)(a) / ISO A.8.22 / CyFun PR.IR-01 / CIS 12 | Zero (config) |
| 4 | F-04 | Cleartext FTP used for production and AI data transfer | DP-03 | Data Protection | 2 | 3 | 6 | High | GDPR 32 / ISO A.8.24 / CyFun PR.DS-02 / CIS 3 | Zero (SFTP/FTPS) |
| 5 | F-05 | Weak RADIUS shared secret and non-unique admin credentials | AC-01 | Access Control | 2 | 3 | 6 | High | NIS2 21(2)(i) / ISO A.8.5 / CyFun PR.AA-01 / CIS 4/5 | Zero (config) |
| 6 | F-06 | Global password encryption disabled on network devices | DP-02 | Data Protection | 2 | 3 | 6 | High | GDPR 32 / NIS2 21(2)(h) / ISO A.8.24 / CyFun PR.DS-01/02 | Zero (config) |
| 7 | F-07 | Port security missing across access-layer switches (5 of 6) | SEG-04 | Segmentation | 2 | 3 | 6 | High | NIS2 21(2)(a) / ISO A.8.22 / CyFun PR.IR-01 / CIS 4 | Zero (config) |
| 8 | F-08 | Centralized logging and service timestamps disabled | LOG-01 | Logging | 3 | 2 | 6 | High | NIS2 21(2)(g) / ISO A.8.15 / CyFun DE.AE-03/PR.PS-04 / CIS 8 | Low (syslog) |
| 9 | F-09 | Remote SSL VPN lacks multi-factor authentication | AC-03 | Access Control | 3 | 2 | 6 | High | NIS2 21(2)(j) / ISO A.8.5 / CyFun PR.AA-03 / CIS 6 | Licensing |
| 10 | F-11 | NTP time synchronization not configured | LOG-02 | Logging | 2 | 2 | 4 | Medium | NIS2 21(2)(g) / ISO A.8.15/8.16 / CyFun DE.CM-01 | Zero (config) |
| 11 | F-12 | Management-plane access not restricted to SSH | DP-01 | Data Protection | 2 | 2 | 4 | Medium | GDPR 32 / NIS2 21(2)(h) / ISO A.8.24 / CyFun PR.DS-02 / CIS 3 | Zero (config) |
| 12 | F-13 | No enforced password complexity policy | AC-02 | Access Control | 2 | 2 | 4 | Medium | NIS2 21(2)(i) / ISO A.8.5 / CyFun PR.AA-01 | Zero (policy) |
| 13 | F-10 | No documented backup/disaster-recovery provisions | GOV-02 | Gov. / Continuity | 1 | 3 | 3 | Medium | NIS2 21(2)(c) / ISO A.5.29/8.13 / CyFun RC.RP-01/PR.DS-11 | Medium (storage) |
| 14 | F-14 | Supplier security criteria omitted from procurement | SUP-01 | Supplier | 2 | 1 | 2 | Low | NIS2 21(2)(d) / ISO A.5.19 / CyFun GV.SC-01/04 | Zero (process) |
| 15 | F-15 | Unassigned policy ownership; incomplete asset inventory | GOV-01 | Governance | 2 | 1 | 2 | Low | NIS2 21(2)(f) / GDPR 30 / ISO A.5.1/5.9 / CyFun ID.AM-01/GV.RR-01 | Zero (governance) |

### Ordering Rationale
* **Equal Scores Ordering:** Within equal scores, the three Critical segmentation findings lead because they compound segmentation failures (allowing a single breach to reach everything).
* **High Scores Ordering:** Among the score-6 findings, standing-exposure items (F-08, F-09) sit just below the foothold-required credential/data items because:
  * **F-01** removes the perimeter filter.
  * **F-02** places public services directly in the trusted core.
  * **F-03** removes internal boundaries.

---

## Top Three Priorities

1. **F-01 (`ALLOW_ALL` Firewall Rule):** The perimeter enforces nothing; the Cisco ASA operates as an unfiltered high-speed pipe. This is a zero-cost configuration fix and stands as the clearest "fix before go-live" requirement.
2. **F-03 (Unbound Inter-VLAN ACLs):** The network segmentation advertised in design documentation is written but left entirely inert. Consequently, Production's AI/R&D data (VLAN 30) is not actually isolated. This exemplifies the documented-versus-implemented gap and requires zero cost to fix.
3. **F-02 (DMZ on Core Switch):** Public-facing services sit inside the trusted core instead of behind a dedicated firewall interface, meaning compromise of the web/FTP server routes straight into the internal network.

*All three priorities are Critical, essentially free to remediate, and must be closed prior to system cutover.*

---

## Scope Statement

* **Audited:** The delivered design of the hub as represented in the assigned Netify dossier (design document, technical & commercial proposal, cost estimation, and the exported device configurations of `Netify_Final.pkt` including the core multilayer switch, six access switches, Cisco ASA edge firewall, and AAA/RADIUS settings).
* **Excluded:** Anything outside the supplied dossier—the live running network, the physical site, endpoint workstations beyond their switch-port configuration, the ISR 4331 router and cable modem (present in BoM/topology but with no configuration supplied), corporate policies not referenced in the documents, and AI systems/products built on top of the network (AI-Act relevant at the entity level, but outside this network audit scope).
* **Audit Type:** Design/documentation ("paper") audit comparing the delivered dossier against applicable regulatory and framework requirements. This is neither a penetration test nor a live-configuration review.

---

## Limitations (per `03_AuditMethod.md §10`)

* **Simulation vs. Reality:** Reviewed documentation and a Packet Tracer simulation rather than a running network; we cannot confirm controls behave as intended in a live environment.
* **Unconfirmed Policy Application:** Could not confirm whether documented policies are applied in practice (e.g., whether password guidance is enforced on real accounts).
* **Unverifiable Operational Processes:** Operational processes such as patching, backup execution, and incident handling were not evidenced; their absence from findings reflects an absence of evidence rather than a confirmed absence of the process itself.
* **Missing Device Configurations:** Some assets have no supplied configuration (ISR 4331, cable modem); findings regarding them rely strictly on design documents.
* **Geographical Assumption:** NIS2 classification rests on coach-provided entity facts; the hub's physical siting is assumed to be Belgium, not independently established from the dossier.
* **Verbal Communication:** Anything conveyed verbally by the design team is non-evidentiary and is excluded from this report.

> **Note:** Every finding traces directly to a control, and every control traces to a regulation established as applicable in Step 1. A finding without a regulatory/framework reference is treated as an opinion rather than an audit result.
