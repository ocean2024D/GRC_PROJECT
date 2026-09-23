# GRC Security & Compliance Audit

## NVIDIA Regional R&D & Production Hub

**Project:** `NVIDIA-REG-RD-2026-1106`  
**Scope:** Regional R&D and Production Hub  
**Assessment:** Security Architecture, Governance, Risk & Compliance (GRC)

This repository contains the GRC and security audit documentation for the **NVIDIA Regional R&D & Production Hub** project.

The assessment reviews the network architecture and technical implementation submitted by **Netify Architecture Group**, using the available design documentation, network analysis, cost estimation, Packet Tracer topology and exported device configurations as evidence.

The objective is to connect regulatory requirements and security controls with technical evidence, identify gaps, document findings and maintain a structured risk register.

---

## Assessment Scope

The assessment covers:

- Network and asset inventory
- VLAN and network segmentation
- Access control and authentication
- Network device management
- Firewall and ACL configuration
- DMZ / application-server exposure
- Logging and monitoring
- Data protection
- VPN security
- Supplier security
- Regulatory applicability
- Security findings
- Risk identification and treatment

The assessed environment contains multiple network zones, including Support, Production, Management, IT, Study, Server Farm, Application/DMZ and Perimeter networks.

---

## Network Environment

The documented environment includes the following VLANs and network segments:

| Zone | VLAN | Subnet | Main Purpose |
|---|---:|---|---|
| Support Sector A | 10 | `192.168.10.0/24` | Support workstations |
| Support Sector B | 20 | `192.168.20.0/24` | Support workstations |
| Production Sector | 30 | `192.168.30.0/24` | R&D, AI and production workloads |
| Management / Secretariat | 40 | `192.168.40.0/24` | Administrative systems |
| IT Department | 50 | `192.168.50.0/24` | IT administration |
| Study Sector | 60 | `192.168.60.0/24` | Training and reference systems |
| Server Farm | 100 | `192.168.100.0/24` | Infrastructure and application services |
| Application / DMZ | 200 | `192.168.200.0/24` | Application / public-facing services |
| Perimeter | — | `10.0.8.0/30` / `10.0.8.4/30` | Firewall and external connectivity |

The Production Sector contains proprietary R&D data, LLM training data/models and software source code and is therefore treated as a particularly sensitive environment within the assessment.

---

## Regulatory & Security Frameworks

The applicability assessment considers the following regulations and standards:

### Applicable / Relevant

- **GDPR**
- **NIS2 / Belgian NIS2 Act**
- **CyFun (CCB)**

### Used as Control References

- **ISO/IEC 27001:2022 Annex A**

### Applicability Requiring Further Determination

- **EU AI Act**
- **ISO/IEC 27001 certification route**

### Outside the Current Scope

- **Cyber Resilience Act (CRA)**
- **DORA**
- **SOC 2**

The regulatory applicability is based on the project assumptions and available documentation. In particular, the physical location of the hub and the exact legal entity characteristics are assumptions that may require confirmation.

---

## Audit Methodology

The audit follows an evidence-based process:

```text
Project Scope
      ↓
Asset & Network Inventory
      ↓
Regulatory Applicability
      ↓
Security Control Checklist
      ↓
Technical Evidence Review
      ↓
Security Findings
      ↓
Risk Assessment
      ↓
Risk Treatment / Remediation
