# Audit Checklist

**NVIDIA Regional R&D & Production Hub**  
**Project:** NVIDIA-REG-RD-2026-1106  
**Purpose:** Design/compliance audit of the Netify Architecture Group submission

## Framework version audited against

CyFun 2025 (aligned to NIST CSF 2.0). Access-control controls use the CSF 2.0 `PR.AA` identifiers; network-segmentation controls use `PR.IR`; supply-chain uses the new `GV.SC` (Govern) category. The retired CSF 1.1 `PR.AC` codes are not used. Stating the version is deliberate — an unversioned CyFun citation is the kind that gets challenged in review.

## Scope

The assigned dossier only — the design document (`Network_Analysis.pdf`), the As-Built proposal + budget (`NVIDIA_Technical_Commercial_Proposal_Final.pdf`, `cost_estimation_.xlsx`), and the exported device configurations (8 config files representing `Netify_Final.pkt`). Nothing outside these three sources.

## Applicable regulations (from Step 1)

- GDPR (personal/HR data)
- Belgian NIS2 Act (important entity — NIS2 Annex II manufacturing, plus group-consolidated size as a wholly-owned NVIDIA subsidiary / linked enterprise under Rec. 2003/361/EC).

Controls are drawn from CyFun 2025, ISO/IEC 27001:2022 Annex A and CIS Benchmarks; NIS2 and GDPR are cited only as the underlying requirement behind each control.

## Rating scale — agreed BEFORE rating

**Risk = Likelihood × Impact**

| Score | Likelihood | Impact |
|---|---|---|
| 3 — High | No control in the way; common technique | Sensitive data, whole sector, or production continuity |
| 2 — Medium | Partial control, or attacker needs a foothold | One sector or one service |
| 1 — Low | Several conditions must align, or insider access | Limited, easily recovered |

| Score | Rating |
|---:|---|
| 9 | Critical |
| 6 | High |
| 3–4 | Medium |
| 1–2 | Low |

> If everything is Critical, nothing is. Anchor Impact in the asset inventory: Production (VLAN 30), Server DMZ (VLAN 100) and IT (VLAN 50) carry the sensitive data and weigh heavier than Study/Support.

# The checklist — 15 checks across 6 areas

| ID | Area | Requirement (regulation) | Control (framework — CyFun 2025 / CSF 2.0) | Yes/No question | Evidence source | Pass condition | Result |
|---|---|---|---|---|---|---|---|
| AC-01 | Access control | NIS2 Art. 21(2)(i) | ISO 27001 A.8.5; CyFun PR.AA-01; CIS 4/5 | Are admin credentials unique and non-default (no shared/default passwords) on the network devices and AAA store? | Core-switch config; server-AAA-configs; firewall config | No default/shared passwords; each admin identity individual | [Step 3] |
| AC-02 | Access control | NIS2 Art. 21(2)(i) | ISO 27001 A.8.5; CyFun PR.AA-01 | Is a minimum password strength / policy defined and enforced anywhere in the dossier? | Design doc SecV; all configs | A password policy (length ≥ 12) is stated AND reflected in config | [Step 3] |
| AC-03 | Access control | NIS2 Art. 21(2)(j) | ISO 27001 A.8.5; CyFun PR.AA-03; CIS 6 | Is MFA required for the internet-facing remote-access (SSL VPN)? | firewall config (webvpn / tunnel-group) | MFA/2FA configured on the VPN, not single-factor local user | [Step 3] |
| SEG-01 | Segmentation | NIS2 Art. 21(2)(a) | ISO 27001 A.8.22; CyFun PR.IR-01; CIS 12 | Are the inter-VLAN isolation ACLs actually applied to an interface (bound with `ip access-group`), not merely defined? | Core-switch config, VLAN SVIs | Each isolation ACL is bound inbound/outbound on its SVI | [Step 3] |
| SEG-02 | Segmentation | NIS2 Art. 21(2)(a) | ISO 27001 A.8.20/8.22; CyFun PR.IR-01 | Does the perimeter firewall enforce a least-privilege inbound rule set (not permit-any)? | firewall config (access-list / access-group) | Rules restrict traffic; no `permit ip … any` catch-all as effective policy | [Step 3] |
| SEG-03 | Segmentation | NIS2 Art. 21(2)(a) | ISO 27001 A.8.22; CyFun PR.IR-01 | Is the DMZ a true isolated zone (separate firewall interface / enforced boundary) as the design claims? | Design doc SecVI; firewall config; topology | DMZ traffic mediated by a firewall interface, not flat inside the core switch | [Step 3] |
| SEG-04 | Segmentation | NIS2 Art. 21(2)(a) | ISO 27001 A.8.22; CyFun PR.IR-01; CIS 4 | Is port-security applied consistently across all access-layer switches as the design states? | All 6 switch configs; design doc SecIII | Port-security present on every access switch, per the documented standard | [Step 3] |
| LOG-01 | Logging & monitoring | NIS2 Art. 21(2)(g) | ISO 27001 A.8.15; CyFun DE.AE-03 / PR.PS-04; CIS 8 | Is centralized/timestamped logging configured on the core and perimeter devices? | Core-switch, firewall configs | A syslog target and/or service timestamps logging is configured | [Step 3] |
| LOG-02 | Logging & monitoring | NIS2 Art. 21(2)(g) | ISO 27001 A.8.15/8.16; CyFun DE.CM-01 | Is time synchronization (NTP) configured so logs are correlatable? | Core-switch, firewall configs | An NTP server is configured on core/perimeter devices | [Step 3] |
| DP-01 | Data protection | GDPR Art. 32; NIS2 Art. 21(2)(h) | ISO 27001 A.8.24; CyFun PR.DS-02; CIS 3 | Is management-plane access encrypted (SSH not Telnet) on the network devices? | All switch configs (line vty); firewall | Transport is SSH; Telnet/unencrypted login not the access method | [Step 3] |
| DP-02 | Data protection | GDPR Art. 32; NIS2 Art. 21(2)(h) | ISO 27001 A.8.24; CyFun PR.DS-01/02 | Are stored device secrets protected (`service password-encryption`, `enable secret`), and shared secrets non-trivial? | All configs; server-AAA-configs | Password encryption on; secrets not plaintext/weak (e.g. not Cisco123) | [Step 3] |
| DP-03 | Data protection | GDPR Art. 32 | ISO 27001 A.8.24; CyFun PR.DS-02; CIS 3 | Is data-in-transit to the file service encrypted (SFTP/FTPS rather than plaintext FTP)? | Design doc SecI/SecVI; core-switch ACLs; topology | File transfer uses an encrypted protocol, not clear-text FTP (port 20/21) | [Step 3] |
| SUP-01 | Supplier security | NIS2 Art. 21(2)(d) | ISO 27001 A.5.19; CyFun GV.SC-01/04 | Does the dossier identify suppliers AND state at least one security requirement/selection criterion on them? | As-Built proposal Sec5 (BoM); cost xlsx | Suppliers named AND ≥ 1 security criterion stated | [Step 3] |
| GOV-01 | Governance | NIS2 Art. 21(2)(f); GDPR Art. 30 | ISO 27001 A.5.1/A.5.9; CyFun ID.AM-01 / GV.RR-01 | Is there a documented asset inventory and a named owner/responsibility for security policy? | Whole dossier | An asset register and a policy owner/governance statement exist | [Step 3] |
| GOV-02 | Governance / continuity | NIS2 Art. 21(2)(c) | ISO 27001 A.5.29/A.8.13; CyFun RC.RP-01 / PR.DS-11 | Is there a documented backup / BC-DR provision for the critical servers? | As-Built proposal; design doc | A backup or BC/DR measure is documented for DNS/AAA/FTP/data | [Step 3] |

## Coverage

- **Access control:** AC-01/02/03
- **Segmentation:** SEG-01/02/03/04
- **Logging:** LOG-01/02
- **Data protection:** DP-01/02/03
- **Supplier:** SUP-01
- **Governance/continuity:** GOV-01/02

**15 checks, 6 areas.** Result column stays blank until Step 3, where every check is recorded — passes included.
