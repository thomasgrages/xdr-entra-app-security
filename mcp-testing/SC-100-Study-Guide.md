# SC-100 Study Guide — Focused on Weak Areas

> **Exam:** Microsoft Cybersecurity Architect (SC-100)  
> **Generated:** February 27, 2026  
> **Purpose:** Targeted remediation based on score report analysis

---

## Score Report Summary

| Section | Weight | Performance |
|---------|--------|-------------|
| Design solutions that align with security best practices and priorities | 20–25% | Medium |
| Design security operations, identity, and compliance capabilities | 25–30% | Medium-High |
| **Design security solutions for infrastructure** | **25–30%** | **Weakest** |
| Design security solutions for applications and data | 20–25% | Strongest |

### Top 3 Skill Gaps (from report)

1. **Specify requirements for securing server and client endpoints**
2. **Design solutions for securing applications**
3. **Design a resiliency strategy for ransomware and other attacks based on Microsoft Security Best Practices**

---

## Priority 1: Specify Requirements for Securing Server and Client Endpoints

### Key Concepts to Master

#### Microsoft Defender for Endpoint
- Onboarding methods: Group Policy, Intune, SCCM, local script, VDI
- Attack Surface Reduction (ASR) rules — understand each rule category
- Endpoint Detection and Response (EDR) — block mode vs. passive mode
- Automated investigation and remediation (AIR)
- Device risk levels and how they integrate with Conditional Access

#### Microsoft Defender for Servers (Defender for Cloud)
- **Plan 1 vs. Plan 2** — know the feature differences:
  - Plan 1: Defender for Endpoint integration, basic CSPM
  - Plan 2: Just-in-time VM access, file integrity monitoring, adaptive application controls, adaptive network hardening, agentless scanning
- Qualys vs. Microsoft Defender Vulnerability Management (MDVM) built-in scanner
- Azure Arc for hybrid/multi-cloud server onboarding

#### Client Endpoint Security
- Microsoft Intune — compliance policies, configuration profiles, endpoint security policies
- Windows Autopilot — deployment scenarios (user-driven, self-deploying, pre-provisioned)
- Application control: Windows Defender Application Control (WDAC) vs. AppLocker
  - **WDAC** is preferred for modern management; works in user and kernel mode
  - **AppLocker** is supplementary; user mode only
- BitLocker and encryption requirements
- Microsoft Defender Application Guard — hardware-based isolation

#### Server Hardening
- Azure Update Manager for patching
- Azure Policy guest configuration for compliance auditing
- Privileged Access Workstations (PAWs) and the Enterprise Access Model (tiers)
- Secure boot, vTPM, and Trusted Launch for Azure VMs
- Confidential computing: DCsv2/DCsv3 VMs, AMD SEV-SNP

### Study Resources
- [Protect endpoints with Defender for Endpoint](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/microsoft-defender-endpoint)
- [Plan Defender for Servers deployment](https://learn.microsoft.com/en-us/azure/defender-for-cloud/plan-defender-for-servers)
- [Attack surface reduction rules reference](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction-rules-reference)
- [Windows Defender Application Control](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/wdac)

### Practice Questions to Focus On
- When to choose Defender for Servers Plan 1 vs. Plan 2
- How to integrate device compliance signals into Zero Trust Conditional Access
- Designing endpoint protection for hybrid (on-prem + Azure + AWS/GCP) environments using Azure Arc
- Choosing between WDAC and AppLocker for different scenarios

---

## Priority 2: Design Solutions for Securing Applications

### Key Concepts to Master

#### Microsoft Defender for Cloud — App Security
- Defender for App Service — threat detection for Azure App Services
- Defender for Key Vault — detecting anomalous access patterns
- Defender for Resource Manager — detecting suspicious management operations
- DevOps security posture in Defender for Cloud (GitHub, Azure DevOps integration)

#### Application Security in the SDLC
- **Microsoft Security Development Lifecycle (SDL)** — threat modeling, static analysis, dynamic analysis
- GitHub Advanced Security:
  - Code scanning (CodeQL)
  - Secret scanning
  - Dependency review / Dependabot
- Azure DevOps: Microsoft Security DevOps extension (SARIF reports)
- Container image scanning in CI/CD pipelines

#### API Security
- Azure API Management security features:
  - OAuth 2.0 / OpenID Connect validation policies
  - Rate limiting and throttling
  - IP filtering
  - Mutual TLS (mTLS)
- Microsoft Defender for APIs (part of Defender for Cloud)
- API inventory and risk assessment

#### Web Application Protection
- Azure Web Application Firewall (WAF):
  - Deployed with Azure Front Door (global) vs. Application Gateway (regional)
  - OWASP Core Rule Set (CRS) — DRS 2.1 managed rules
  - Custom rules and exclusions
  - WAF policy modes: Detection vs. Prevention
- Azure DDoS Protection: Network vs. IP Protection SKUs
- Azure Front Door + Private Link for origin protection

#### Application Authentication & Authorization
- Microsoft Identity Platform (MSAL)
- App registrations, service principals, and managed identities
  - **Always prefer managed identities** over service principals with secrets
- Workload identity federation — for GitHub Actions, Kubernetes, external IdPs
- API permissions: Delegated vs. Application; admin consent requirements
- Continuous Access Evaluation (CAE) for apps

### Study Resources
- [Secure your applications in Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-app-service-introduction)
- [Azure WAF on Front Door](https://learn.microsoft.com/en-us/azure/web-application-firewall/afds/afds-overview)
- [Microsoft Security Development Lifecycle](https://www.microsoft.com/en-us/securityengineering/sdl)
- [Workload identity federation](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation)

### Practice Questions to Focus On
- When to use WAF on Front Door vs. Application Gateway
- Designing DevSecOps pipelines with security scanning at each stage
- Choosing managed identity vs. service principal vs. workload identity federation
- Protecting APIs exposed to partners vs. internal consumers

---

## Priority 3: Design a Resiliency Strategy for Ransomware and Other Attacks

### Key Concepts to Master

#### Microsoft Ransomware Best Practices (CRITICAL)
This is directly from the [Microsoft Security Best Practices](https://learn.microsoft.com/en-us/security/ransomware/) — study this thoroughly.

**Three phases of ransomware protection:**
1. **Prepare** — make your organization harder to attack
2. **Limit** — limit the scope of damage
3. **Respond** — make it hard to operate without paying

#### Backup and Recovery Strategy
- **Azure Backup:** vault redundancy (LRS, ZRS, GRS), soft delete, multi-user authorization (MUA)
- **Immutable vaults** — cannot be disabled once set to locked mode; prevents ransomware from deleting backups
- **Azure Site Recovery (ASR)** — business continuity / disaster recovery
- 3-2-1 backup strategy: 3 copies, 2 different media, 1 offsite
- Isolated recovery environment (clean room recovery)
- Cross-region and cross-subscription backup copies

#### Privileged Access Protection
- **Privileged Access Strategy** (Microsoft's Enterprise Access Model):
  - Control Plane → Management Plane → Data/Workload Plane
- Privileged Access Workstations (PAWs)
- Just-in-Time (JIT) access — Microsoft Entra PIM
- Emergency access (break-glass) accounts
- Lateral movement prevention — credential isolation, Local Admin Password Solution (LAPS)

#### Network Segmentation to Limit Blast Radius
- Microsegmentation with Azure Firewall / NSGs / ASGs
- Zero Trust network design — verify explicitly, least privilege, assume breach
- Hub-spoke topology with Azure Firewall for east-west traffic inspection
- Azure Private Link / Private Endpoints to eliminate public exposure

#### Identity-Based Attack Resilience
- Microsoft Entra ID Protection — risk-based Conditional Access
- On-prem AD: Defender for Identity — detecting pass-the-hash, pass-the-ticket, DCSync
- Protect identity synchronization: secure Microsoft Entra Connect / Cloud Sync servers
- Phishing-resistant MFA: FIDO2 keys, Windows Hello for Business, certificate-based auth
- Token protection and token theft detection

#### Incident Response & Detection
- Microsoft Sentinel — analytics rules, playbooks (Logic Apps), SOAR
- Microsoft Defender XDR — unified incident queue, automated investigation
- Microsoft Secure Score — track and improve security posture
- Attack simulation training in Defender for Office 365

#### Key Frameworks
- **NIST Cybersecurity Framework** — Identify, Protect, Detect, Respond, Recover
- **MITRE ATT&CK** — understand tactics relevant to ransomware:
  - Initial Access → Execution → Lateral Movement → Exfiltration → Impact
- **Microsoft Cloud Adoption Framework (CAF) — Security pillar**

### Study Resources
- [Human-operated ransomware — Microsoft guidance](https://learn.microsoft.com/en-us/security/ransomware/human-operated-ransomware)
- [Backup and restore plan for ransomware](https://learn.microsoft.com/en-us/security/ransomware/protect-against-ransomware-phase2)
- [Privileged access strategy](https://learn.microsoft.com/en-us/security/privileged-access-workstations/privileged-access-strategy)
- [Microsoft Entra PIM](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure)

### Practice Questions to Focus On
- Designing immutable backup strategies across hybrid environments
- Choosing the right privileged access controls for different admin roles
- Architecting network segmentation to contain a ransomware incident
- Planning a recovery strategy after a ransomware attack (order of operations)

---

## Secondary Focus: Design Security Solutions for Infrastructure

This was your **weakest overall section** by the bar chart. The topics above (endpoints, resiliency) overlap heavily with infrastructure. Additionally, study:

### Network Security
- Azure Firewall (Standard vs. Premium) — IDPS, TLS inspection, URL filtering (Premium only)
- Azure Firewall Manager — managing policies across multiple firewalls
- Network Virtual Appliances (NVAs) vs. Azure-native solutions
- Azure DDoS Protection integration with virtual networks
- ExpressRoute security: private peering encryption, Microsoft peering filters

### Hybrid & Multi-Cloud Security
- Azure Arc — extend Azure security to on-prem, AWS, GCP
  - Arc-enabled servers, Kubernetes, SQL, and data services
  - Apply Azure Policy, Defender for Cloud, and monitoring across clouds
- Microsoft Defender for Cloud multi-cloud connectors (AWS, GCP)
- CSPM (Cloud Security Posture Management) — foundational vs. Defender CSPM
  - Attack path analysis
  - Cloud security explorer

### Container & Kubernetes Security
- Microsoft Defender for Containers
  - Runtime threat detection
  - Vulnerability assessment for container images
  - Kubernetes admission control (Azure Policy for AKS)
- AKS network policies — Azure CNI vs. kubenet
- Workload identity for AKS pods (replaces pod-managed identity)
- Private AKS clusters and authorized IP ranges

### Key Vault & Secrets Management
- Azure Key Vault: Premium (HSM-backed) vs. Standard
- Managed HSM and Azure Dedicated HSM — when to use each
- Key rotation policies and automation
- RBAC vs. access policies for Key Vault
- Certificate management and auto-renewal

### Study Resources
- [Azure Firewall Premium features](https://learn.microsoft.com/en-us/azure/firewall/premium-features)
- [Azure Arc overview](https://learn.microsoft.com/en-us/azure/azure-arc/overview)
- [Defender for Containers](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-containers-introduction)
- [Defender CSPM](https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-cloud-security-posture-management)

---

## Study Plan (Recommended 4-Week Schedule)

### Week 1: Ransomware Resiliency & Backup Strategy
- [ ] Read Microsoft ransomware best practices (all 3 phases)
- [ ] Study Azure Backup immutable vaults and MUA
- [ ] Study privileged access strategy and Enterprise Access Model
- [ ] Practice: Design a ransomware recovery architecture for a hybrid org
- [ ] Review NIST CSF and MITRE ATT&CK ransomware tactics

### Week 2: Endpoint Security (Servers & Clients)
- [ ] Study Defender for Endpoint — onboarding, ASR, EDR
- [ ] Study Defender for Servers Plan 1 vs. Plan 2 in depth
- [ ] Study WDAC vs. AppLocker decision matrix
- [ ] Study Azure Arc for multi-cloud server management
- [ ] Practice: Design endpoint strategy for hybrid org with Windows/Linux servers + Windows/Mac clients

### Week 3: Infrastructure Security (Weakest Section)
- [ ] Study Azure Firewall Standard vs. Premium
- [ ] Study container security: Defender for Containers, AKS hardening
- [ ] Study Defender CSPM: attack paths, security explorer
- [ ] Study Key Vault architecture decisions
- [ ] Practice: Design network security for hub-spoke with multi-cloud

### Week 4: Application Security & Review
- [ ] Study DevSecOps pipeline security (GitHub Advanced Security, Defender for DevOps)
- [ ] Study WAF deployment strategies (Front Door vs. App Gateway)
- [ ] Study managed identities and workload identity federation
- [ ] Full practice exam review
- [ ] Focus remaining time on lowest-confidence topics

---

## Quick-Reference: Key Decision Points for SC-100

| Scenario | Solution |
|----------|----------|
| Protect VMs from ransomware backup deletion | Immutable Azure Backup vault + MUA |
| Secure admin access to Azure portal | PIM + Conditional Access + PAW |
| Multi-cloud security posture visibility | Defender for Cloud + CSPM + Arc |
| Protect web apps from OWASP Top 10 | Azure WAF (on Front Door or App GW) |
| Scan containers for vulnerabilities | Defender for Containers |
| Restrict which apps can run on endpoints | WDAC (preferred) or AppLocker |
| Secure east-west traffic in Azure | Azure Firewall + NSGs + microsegmentation |
| Detect lateral movement in on-prem AD | Microsoft Defender for Identity |
| Phishing-resistant MFA | FIDO2 / Windows Hello for Business |
| Secure secrets for applications | Key Vault + Managed Identity |
| Protect against DDoS | Azure DDoS Protection + WAF |
| Multi-cloud server management | Azure Arc + Defender for Cloud |

---

*Good luck on your retake! Focus your study time proportional to the gap sizes — infrastructure and the three priority skills listed above should get the most attention.*
