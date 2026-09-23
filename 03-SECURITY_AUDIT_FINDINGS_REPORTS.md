# SECURITY AUDIT FINDINGS REPORT
## Project NVIDIA-REG-RD-2026-1106
### NVIDIA Regional R&D & Production Hub

**Frameworks & Scope:** CyFun 2025 / NIST CSF 2.0, GDPR, Belgian NIS2 Act (Manufacturing Sector)  
**Scope Documents:** Design Document, Technical Proposal, and Device Configuration Files

---

## Findings Summary

| ID | Title | Related Check | Rating | $L \times I$ |
| :--- | :--- | :--- | :--- | :--- |
| F-01 | Overly Permissive Inbound Edge Firewall Policy (`ALLOW_ALL`) | SEG-02 | Critical | $3 \times 3 = 9$ |
| F-02 | DMZ Server Farm Terminated on Core Switch SVI Instead of Perimeter Firewall | SEG-03 | Critical | $3 \times 3 = 9$ |
| F-03 | Defined Inter-VLAN Access Control Lists Not Bound to Switch Virtual Interfaces | SEG-01 | Critical | $3 \times 3 = 9$ |
| F-04 | Cleartext FTP Protocol Used for Production and AI Data Transfer | DP-03 | High | $2 \times 3 = 6$ |
| F-05 | Weak RADIUS Shared Secret and Non-Unique Admin Credentials | AC-01 | High | $2 \times 3 = 6$ |
| F-06 | Global Password Encryption Disabled on Network Devices | DP-02 | High | $2 \times 3 = 6$ |
| F-07 | Port Security Missing Across Access-Layer Switches | SEG-04 | High | $2 \times 3 = 6$ |
| F-08 | Centralized Logging and Service Timestamps Disabled | LOG-01 | High | $3 \times 2 = 6$ |
| F-09 | Remote SSL VPN Lacks Multi-Factor Authentication (MFA) | AC-03 | High | $3 \times 2 = 6$ |
| F-10 | Absence of Documented Backup and Disaster Recovery Provisions | GOV-02 | Medium | $1 \times 3 = 3$ |
| F-11 | Network Time Protocol (NTP) Synchronization Not Configured | LOG-02 | Medium | $2 \times 2 = 4$ |
| F-12 | Management-Plane Access Not Restricted to SSH | DP-01 | Medium | $2 \times 2 = 4$ |
| F-13 | Absence of Enforced Password Complexity Policy | AC-02 | Medium | $2 \times 2 = 4$ |
| F-14 | Supplier Security Criteria Omitted from Procurement Documentation | SUP-01 | Low | $2 \times 1 = 2$ |
| F-15 | Unassigned Policy Ownership and Incomplete Formal Asset Inventory | GOV-01 | Low | $2 \times 1 = 2$ |

* **Critical:** 3
* **High:** 6
* **Medium:** 4
* **Low:** 2

---

## Detailed Findings

### F-01: Overly Permissive Inbound Edge Firewall Policy (`ALLOW_ALL`)
* **Related Check:** SEG-02
* **Rating:** Critical ($L \times I = 9$)
* **Observation:** The Cisco ASA perimeter firewall configuration defines and applies an unrestricted inbound access list to the inside interface: `access-list ALLOW_ALL extended permit ip 192.168.100.0 255.255.255.0 any` and `access-group ALLOW_ALL in interface inside`. This rule permits all IP traffic originating from the internal server network to traverse without stateful inspection or least-privilege filtering.
* **Evidence:** `firewall-config.txt`, lines 39-44.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.20/8.22; CyFun 2025 PR.IR-01. Underlying requirement: Belgian NIS2 Act/NIS2 Art. 21(2)(a).
* **Risk:** NVIDIA's central AAA/RADIUS server (`192.168.100.4`) and DNS/DHCP server (`192.168.100.3`) within VLAN 100 are exposed to unfiltered lateral and external threats. A breach in any internal sector will bypass perimeter defense entirely, directly compromising the high-performance AI production infrastructure in VLAN 30 and critical intellectual property (IP) data.
* **Recommendation:** Replace the `ALLOW_ALL` access group and rule set on the Cisco ASA firewall with granular, stateful inspection rules that strictly permit only required business protocols.
* **Priority:** Immediate (pre-cutover prerequisite with zero hardware cost, well within the 300,000 EUR budget ceiling).

---

### F-02: DMZ Server Farm Terminated on Core Switch SVI Instead of Perimeter Firewall
* **Related Check:** SEG-03
* **Rating:** Critical ($L \times I = 9$)
* **Observation:** The Application and FTP server residing in the DMZ (VLAN 200, subnet `192.168.200.0/24`) are physically and logically connected to the central Cisco Catalyst 3560 Multilayer Switch (`interface GigabitEthernet1/0/9 switchport access vlan 200`) rather than terminating on a dedicated physical interface of the Cisco ASA edge firewall.
* **Evidence:** `Network_Analysis.pdf` §VI, pp. 6-7 / `core-switch-config.txt`, lines 52-54.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.22; CyFun 2025 PR.IR-01. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(a).
* **Risk:** Public-facing services requiring internet accessibility bypass hardware firewall boundary mediation. Compromise of the DMZ application server grants direct Layer 3 routing access into the internal core network via the multilayer switch SVIs, completely invalidating the security isolation expected of a true Demilitarized Zone.
* **Recommendation:** Re-architect the perimeter topology to physically or logically terminate the DMZ segment on a dedicated interface (e.g., `GigabitEthernet1/3`) of the Cisco ASA 5506-X firewall.
* **Priority:** Critical (pre-cutover architecture change, utilizes existing hardware capacity within budget).

---

### F-03: Defined Inter-VLAN Access Control Lists Not Bound to Switch Virtual Interfaces
* **Related Check:** SEG-01
* **Rating:** Critical ($L \times I = 9$)
* **Observation:** Extended Access Control Lists (`ACL-SUPPORT1-IN`, `ACL-PRODUCTION-IN`, `ACL-MANAGEMENT-IN`, `ACL-STUDY-IN`, `ACL-IT-IN`) are fully defined in the core multilayer switch configuration. However, an examination of the corresponding SVI interface configurations (`interface Vlan10`, `Vlan30`, `Vlan40`, etc.) reveals that no `ip access-group` command is applied to bind these rules inbound or outbound on the respective subnets.
* **Evidence:** `core-switch-config.txt`, lines 65-150.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.22; CyFun 2025 PR.IR-01; CIS Controls v8 Control 12. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(a).
* **Risk:** Although the filtering logic is written in the device configuration, it remains entirely inactive. Consequently, zero-trust departmental segmentation between functional sectors—specifically protecting the Production high-end AI workstations (VLAN 30) from standard support and management VLANs—is not enforced at Layer 3, permitting unrestricted lateral movement across internal subnets.
* **Recommendation:** Bind each departmental ACL to its respective SVI on the core multilayer switch using the appropriate command syntax (e.g., `ip access-group ACL-PRODUCTION-IN in`).
* **Priority:** Immediate (zero-cost configuration fix with no hardware dependency).

---

### F-04: Cleartext FTP Protocol Used for Production and AI Data Transfer
* **Related Check:** DP-03
* **Rating:** High ($L \times I = 6$)
* **Observation:** The network topology, design documentation, and core switch ACL rules indicate that file transfers to and from the file service utilize unencrypted standard FTP (TCP ports 20 and 21) rather than a secure alternative such as SFTP or FTPS.
* **Evidence:** `Network_Analysis.pdf` §I/§VI, `core-switch-config.txt`, ACL port rules.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.24; CyFun 2025 PR.DS-02; CIS Controls v8 Control 3. Underlying requirement: GDPR Art. 32.
* **Risk:** Sensitive production datasets, R&D files, and AI model weights transmitted over the internal network are exposed to cleartext sniffing and session hijacking. Because data traffic traverses shared enterprise segments (VLAN 30), any compromised endpoint can capture file transfer credentials and payload contents in transit.
* **Recommendation:** Disable cleartext FTP (ports $20/21$) across all devices and enforce SFTP or FTPS with TLS encryption for all file service communications.
* **Priority:** High, scheduled before production data loading, zero extra hardware cost.

---

### F-05: Weak RADIUS Shared Secret and Non-Unique Admin Credentials
* **Related Check:** AC-01
* **Rating:** High ($L \times I = 6$)
* **Observation:** Device configurations and AAA server settings reveal the use of weak, predictable shared secrets (e.g., `Cisco123`) and shared administrative identities rather than strictly individualized credentials mapped to separate accounts.
* **Evidence:** `core-switch-config.txt`, `server-AAA-configs`, `firewall-config.txt`.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.5; CyFun 2025 PR.AA-01; CIS Controls v8 Controls 4/5. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(i).
* **Risk:** The IT Department (VLAN 50) and core network management plane are vulnerable to dictionary attacks and credential guessing. A single compromised shared secret grants administrative root access across all network infrastructure devices simultaneously, undermining accountability.
* **Recommendation:** Replace weak shared secrets with strong, complex cryptographic keys (minimum 16 random characters) and provision unique, individual administrator accounts mapped to the AAA/RADIUS store.
* **Priority:** High, zero cost.

---

### F-06: Global Password Encryption Disabled on Network Devices
* **Related Check:** DP-02
* **Rating:** High ($L \times I = 6$)
* **Observation:** An inspection of the device configuration files shows that global password encryption is explicitly disabled (`no service password-encryption`), leaving local enable secrets and user passwords stored in plaintext or weak reversible formats within the text configuration files.
* **Evidence:** All switch configs (line vty), `firewall-config.txt`.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.24; CyFun 2025 PR.DS-01/02. Underlying requirement: GDPR Art. 32; Belgian NIS2 Act / NIS2 Art. 21(2)(h).
* **Risk:** Any individual with read access to configuration backups, text files, or console logs can instantly extract device administrative secrets, enabling immediate privilege escalation across the entire IT infrastructure (VLAN 50).
* **Recommendation:** Enable global password encryption (`service password-encryption`) and update all enable secrets to strong cryptographic hashes.
* **Priority:** High, zero cost.

---

### F-07: Port Security Missing Across Access-Layer Switches
* **Related Check:** SEG-04
* **Rating:** High ($L \times I = 6$)
* **Observation:** An audit of all six access-layer switch configurations against the design document specification shows that port-security (such as maximum MAC address limits, sticky learning, and shutdown violation mode) is omitted from 5 out of 6 switches, leaving access ports unhardened.
* **Evidence:** All 6 switch configs, `Network_Analysis.pdf` §III.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.22; CyFun 2025 PR.IR-01; CIS Controls v8 Control 4. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(a).
* **Risk:** Physical drop cables in office and production areas (VLAN 30 and VLAN 50) can be unplugged by unauthorized personnel or visitors to connect rogue laptops, enabling direct layer-2 bridging and network reconnaissance without detection.
* **Recommendation:** Implement consistent port-security configurations (restricting max MAC addresses and setting violation shutdown) across all access ports on every access switch per the documented design standard.
* **Priority:** High, zero cost.

---

### F-08: Centralized Logging and Service Timestamps Disabled
* **Related Check:** LOG-01
* **Rating:** High ($L \times I = 6$)
* **Observation:** The core switch and perimeter firewall configurations lack any target configuration for an external centralized syslog server. Furthermore, service timestamps for log entries are explicitly disabled (`no service timestamps log`), preventing accurate chronological event tracing.
* **Evidence:** `core-switch-config.txt`, `firewall-config.txt`.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.15; CyFun 2025 DE.AE-03 / PR.PS-04; CIS Controls v8 Control 8. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(g).
* **Risk:** Security incidents, unauthorized configuration changes, or intrusion attempts cannot be centrally monitored, correlated, or preserved. Local volatile log buffers will overwrite quickly, destroying forensic evidence required for NIS2 compliance incident reporting.
* **Recommendation:** Enable service timestamps globally and configure a designated centralized syslog server target on all core and perimeter devices.
* **Priority:** High, zero cost.

---

### F-09: Remote SSL VPN Lacks Multi-Factor Authentication (MFA)
* **Related Check:** AC-03
* **Rating:** High ($L \times I = 6$)
* **Observation:** The perimeter firewall configuration for remote SSL VPN access (`tunnel-group` and `webvpn` settings) relies exclusively on single-factor local user authentication (`vpnuser`), with no multi-factor authentication (MFA) mechanism integrated.
* **Evidence:** `firewall-config.txt`.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.5; CyFun 2025 PR.AA-03; CIS Controls v8 Control 6. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(j).
* **Risk:** External remote workers or threat actors who capture or brute-force a single VPN credential gain unhindered network-level entry directly into the enterprise perimeter, bypassing physical security controls.
* **Recommendation:** Integrate the remote-access VPN tunnel groups with an external RADIUS/MFA provider (e.g., Duo or token-based 2FA) to enforce multi-factor authentication.
* **Priority:** High, licensing cost planned within the 300,000 EUR budget ceiling.

---

### F-10: Absence of Documented Backup and Disaster Recovery Provisions
* **Related Check:** GOV-02
* **Rating:** Medium ($L \times I = 3$)
* **Observation:** The As-Built proposal and network design documentation contain no formal provisions, schedules, or architectural plans for automated backups, data replication, or Business Continuity / Disaster Recovery (BC-DR) procedures for critical servers (DNS, AAA, FTP).
* **Evidence:** `NVIDIA_Technical_Commercial_Proposal_Final.pdf`, `Network_Analysis.pdf`.
* **Reference:** ISO/IEC 27001:2022 Annex A 5.29/8.13; CyFun 2025 RC.RP-01/PR.DS-11. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(c).
* **Risk:** In the event of hardware failure, corruption, or ransomware attack affecting critical infrastructure servers, recovery will be unverified and prolonged, resulting in severe disruption to production continuity (VLAN 30).
* **Recommendation:** Formulate, document, and test an automated backup and disaster recovery procedure for all core servers, ensuring offsite or secondary storage replication.
* **Priority:** Medium, software/storage costs accounted for in the 300,000 EUR envelope.

---

### F-11: Network Time Protocol (NTP) Synchronization Not Configured
* **Related Check:** LOG-02
* **Rating:** Medium ($L \times I = 4$)
* **Observation:** Configuration files for the core switch and perimeter firewall do not contain any `ntp server` configuration commands, meaning device clocks operate independently without synchronized time sources.
* **Evidence:** `core-switch-config.txt`, `firewall-config.txt`.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.15/8.16; CyFun 2025 DE.CM-01. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(g).
* **Risk:** Log entries across disparate network devices will exhibit time drift, making chronological correlation during cross-device forensic investigations and incident response virtually impossible.
* **Recommendation:** Configure reliable internal NTP server sources on the core multilayer switch and perimeter firewall.
* **Priority:** Medium, zero cost.

---

### F-12: Management-Plane Access Not Restricted to SSH
* **Related Check:** DP-01
* **Rating:** Medium ($L \times I = 4$)
* **Observation:** Review of the VTY line configurations (`line vty 0 4`) across network switches indicates that management transport protocol is not strictly restricted to SSH (`transport input ssh` is missing or permits unencrypted fallback).
* **Evidence:** All switch configs (line vty), `firewall-config.txt`.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.24; CyFun 2025 PR.DS-02; CIS Controls v8 Control 3. Underlying requirement: GDPR Art. 32; Belgian NIS2 Act / NIS2 Art. 21(2)(h).
* **Risk:** Administrators connecting to network devices over the local network risk exposing management commands and administrative credentials in cleartext if unencrypted protocols are permitted.
* **Recommendation:** Explicitly configure `transport input ssh` on all VTY lines across every network device and disable legacy management protocols.
* **Priority:** Medium, zero cost.

---

### F-13: Absence of Enforced Password Complexity Policy
* **Related Check:** AC-02
* **Rating:** Medium ($L \times I = 4$)
* **Observation:** Section V of the design document mentions general password guidelines, but no explicit minimum length requirement of at least 12 characters is defined or enforced programmatically across device configurations or AAA policies.
* **Evidence:** Design doc §V, all configs.
* **Reference:** ISO/IEC 27001:2022 Annex A 8.5; CyFun 2025 PR.AA-01. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(i).
* **Risk:** Users or administrators may select short, easily guessable passwords, increasing vulnerability to brute-force and password-spraying attacks against administrative portals.
* **Recommendation:** Document a formal password complexity policy mandating a minimum length of 12 characters with character diversity, and enforce it via AAA server configuration rules.
* **Priority:** Medium, zero cost.

---

### F-14: Supplier Security Criteria Omitted from Procurement Documentation
* **Related Check:** SUP-01
* **Rating:** Low ($L \times I = 2$)
* **Observation:** The As-Built proposal (§5 BoM) and cost estimation spreadsheet identify hardware and service suppliers (e.g., Cisco, Dell) by name and cost, but include no baseline security requirements, compliance verifications, or selection criteria for third-party vendors.
* **Evidence:** `NVIDIA_Technical_Commercial_Proposal_Final.pdf` §5, `cost_estimation.xlsx`.
* **Reference:** ISO/IEC 27001:2022 Annex A 5.19; CyFun 2025 GV.SC-01/04. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(d).
* **Risk:** Supply-chain vulnerabilities introduced via unvetted third-party hardware or service providers could compromise the integrity of the NVIDIA regional hub infrastructure without contractual recourse.
* **Recommendation:** Define and append baseline security evaluation criteria and compliance clauses to future vendor procurement and vendor management agreements.
* **Priority:** Low, zero financial impact on the 300,000 EUR ceiling.

---

### F-15: Unassigned Policy Ownership and Incomplete Formal Asset Inventory
* **Related Check:** GOV-01
* **Rating:** Low ($L \times I = 2$)
* **Observation:** While technical equipment lists exist within the dossier, a formal, comprehensive asset inventory register mapped to classification levels and a designated named owner for security policy governance are absent.
* **Evidence:** Whole dossier review.
* **Reference:** ISO/IEC 27001:2022 Annex A 5.1/5.9; CyFun 2025 ID.AM-01 / GV.RR-01. Underlying requirement: Belgian NIS2 Act / NIS2 Art. 21(2)(f); GDPR Art. 30.
* **Risk:** Lack of clear asset ownership and formal inventory tracking leads to accountability gaps during security incidents, making it difficult to determine responsibility for maintaining and patching specific system components.
* **Recommendation:** Publish a formal asset inventory register in alignment with GDPR Art. 30 and assign a named Information Security Officer as the accountable policy owner.
* **Priority:** Low, zero cost.
