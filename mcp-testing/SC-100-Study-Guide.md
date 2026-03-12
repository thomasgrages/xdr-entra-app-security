# SC-100 Study Guide — Focused Retake Preparation

> **Exam:** Microsoft Cybersecurity Architect (SC-100)  
> **Last Attempt:** March 4, 2026 — Score: **623 / 700** (77 points short)  
> **Updated:** March 4, 2026  
> **Purpose:** Deep-dive remediation for retake — second attempt strategy

---

## Score Report Analysis (March 4, 2026 Attempt)

| Section | Weight | Est. Score | Gap to Passing | Priority |
|---------|--------|-----------|----------------|----------|
| **Design security solutions for infrastructure** | **25–30%** | **~50%** | **Large** | **🔴 CRITICAL** |
| Design solutions that align with security best practices and priorities | 20–25% | ~65% | Medium | 🟡 HIGH |
| Design security operations, identity, and compliance capabilities | 25–30% | ~65% | Medium | 🟡 HIGH |
| Design security solutions for applications and data | 20–25% | ~75% | Small | 🟢 MAINTAIN |

### What This Means

Infrastructure is the **highest-weight section AND your weakest area** — fixing this alone could swing 25+ points. The two medium sections each need 5-10% improvement. Applications/data is near passing on its own — maintain but don't over-invest.

### Revised Top Focus Areas

1. **Design security solutions for infrastructure** — needs the most improvement (~50% → 75%+)
2. **Security best practices, Zero Trust, and ransomware resiliency** — needs solid improvement (~65% → 75%+)
3. **Security operations, identity governance, and compliance** — needs moderate improvement (~65% → 75%+)
4. **Application and data security** — maintain current level

---

## 🔴 CRITICAL — Design Security Solutions for Infrastructure (25–30%)

**This is your #1 priority. Highest exam weight + biggest gap = maximum point recovery.**

### 1. Design a Strategy for Securing Server and Client Endpoints

#### Microsoft Defender for Endpoint (MDE)
- **Onboarding methods** and when to use each:
  - Intune (cloud-managed), Group Policy (domain-joined), SCCM/ConfigMgr (hybrid), local script (testing), VDI (non-persistent)
- **Attack Surface Reduction (ASR) rules** — 16+ rule categories:
  - Block Office apps from creating child processes
  - Block credential stealing from LSASS
  - Block executable content from email/webmail
  - Block untrusted/unsigned processes from USB
  - Use audit mode first → then block mode with exclusions
- **EDR in block mode** — acts as primary AV when non-Microsoft AV is installed
- **Automated Investigation and Remediation (AIR)** — triage alerts, automatic remediation actions
- **Device risk levels** (Low/Medium/High/Critical) → feed into Conditional Access compliance policies
- **Live Response** — remote investigation shell on devices
- **Indicators of Compromise (IoC)** — file hash, IP, URL, certificate-based blocking

#### Microsoft Defender for Servers (Defender for Cloud)
| Feature | Plan 1 | Plan 2 |
|---------|--------|--------|
| MDE integration | ✅ | ✅ |
| Vulnerability assessment (MDVM) | ✅ | ✅ |
| Just-in-time VM access | ❌ | ✅ |
| File integrity monitoring | ❌ | ✅ |
| Adaptive application controls | ❌ | ✅ |
| Adaptive network hardening | ❌ | ✅ |
| Docker host hardening | ❌ | ✅ |
| Agentless secret scanning | ❌ | ✅ |
| Agentless malware scanning | ❌ | ✅ |

- **Key exam scenario:** "A company needs JIT access to VMs and file integrity monitoring" → **Plan 2**
- **Key exam scenario:** "Minimize cost, only need endpoint protection for servers" → **Plan 1**

#### Client Endpoint Security (Intune)
- **Compliance policies** — define what "compliant" means (OS version, encryption, no jailbreak)
- **Configuration profiles** — push settings (firewall, certificates, Wi-Fi, VPN)
- **Endpoint security policies** — antivirus, disk encryption, firewall, EDR, ASR, account protection
- **Security baselines** — preconfigured templates for Windows, Edge, Defender for Endpoint, Microsoft 365 Apps
- **Conditional Access integration:** Device must be compliant + user risk = low → grant access

#### Application Control — Deep Dive (CRITICAL exam topic)

##### WDAC vs. AppLocker — Full Comparison

| Feature | WDAC | AppLocker |
|---------|------|-----------|
| Enforcement scope | **Kernel mode + user mode** | User mode only |
| Blocks drivers/kernel code | ✅ | ❌ |
| Management options | Intune, GPO, SCCM, MDM, PowerShell | GPO only |
| Managed installer support | ✅ | ❌ |
| Intelligent Security Graph (ISG) | ✅ | ❌ |
| Windows S mode support | ✅ | ❌ |
| Tamper resistant | ✅ (kernel-level, can't be bypassed by local admin) | ❌ (user-mode, local admin can disable) |
| Per-user rules | ❌ (device-wide only) | ✅ |
| File types controlled | PE (.exe, .dll, .sys), scripts, MSI, packaged apps, COM objects | .exe, .dll, scripts, MSI, packaged apps |
| **Microsoft recommendation** | **Preferred — primary control** | Supplementary only |

**Key architectural difference:** WDAC operates as a **kernel-mode code integrity policy** enforced by the Windows kernel itself. Even a local administrator cannot bypass it. AppLocker runs as a **user-mode service** (AppIDSvc) — a local admin can stop the service or modify policies.

##### WDAC Policy Architecture

**Policy types:**
- **Base policy** — the primary allow/deny list (every system needs one)
- **Supplemental policies** — extend a base policy with additional rules WITHOUT editing the base
  - Example: IT deploys a restrictive base policy; a team adds a supplemental policy for their LOB app
  - Requires base policy to have `AllowSupplementalPolicies` option enabled
  - **Exam tip:** supplemental policies can ONLY add allow rules, never deny

**Policy modes:**
- **Audit mode** — logs what WOULD be blocked (Event ID 3076) but allows everything
  - **Always start here** — deploy audit mode first, analyze logs, then enforce
- **Enforced mode** — actually blocks unauthorized code (Event ID 3077)
- **Both modes log to:** `Applications and Services Logs > Microsoft > Windows > CodeIntegrity > Operational`

**Trust authorization options (how WDAC decides what's allowed):**

| Rule Level | What It Trusts | Security Level | Maintenance |
|------------|---------------|----------------|-------------|
| **Hash** | Exact file hash | Highest | Breaks on every update |
| **FileName** | Original filename (PE header) | Medium | Survives renames |
| **FilePath** | Files in specific directories | Lower | Easy but less secure |
| **Publisher** | Code signing certificate + publisher | Recommended | Survives updates |
| **FilePublisher** | Publisher + product + version (≥) | Best balance | Version-aware |
| **LeafCertificate** | Specific signing certificate | High | Cert-specific |
| **PcaCertificate** | Root/intermediate CA | Broadest | Trusts entire CA chain |
| **Intelligent Security Graph (ISG)** | Microsoft's cloud reputation | Convenient | Requires cloud connectivity |
| **Managed installer** | Anything deployed by SCCM/Intune | Convenient | Trusts your deployment tool |

**Exam-critical rule levels:**
- **FilePublisher** = Microsoft's recommended default for most orgs (balances security + maintainability)
- **Hash** = most restrictive but impractical at scale (changes every update)
- **ISG** = allows apps with good cloud reputation — useful for user-driven environments but trusts Microsoft's judgment
- **Managed installer** = trusts apps deployed through your management tool (Intune/SCCM) — great for enterprise

##### WDAC Deployment with Intune

1. Create WDAC policy using the **WDAC Wizard** (GUI tool) or PowerShell (`New-CIPolicy`)
2. Convert to binary: `ConvertFrom-CIPolicy -XmlFilePath policy.xml -BinaryFilePath policy.bin`
3. Deploy via Intune: **Endpoint security > Application control** OR **Configuration profiles > Custom OMA-URI**
4. Intune-native application control profile supports:
   - Trust apps from Microsoft Intelligent Security Graph
   - Trust apps from managed installer (Intune itself)
   - Custom WDAC policy upload (.bin file)

**Smart App Control (Windows 11):**
- Consumer-oriented feature that uses ISG + AI to block untrusted apps
- Starts in **evaluation mode** → transitions to **enforced** or **off** automatically
- **Not manageable by enterprise tools** — cannot be re-enabled once turned off
- **Exam tip:** Smart App Control is NOT the answer for enterprise scenarios — WDAC is

##### AppLocker — When It Still Matters

AppLocker is **not deprecated** but is positioned as supplementary. Scenarios where AppLocker appears on the exam:

1. **Per-user rules needed** — WDAC is device-wide only; AppLocker can target specific users/groups
   - Example: "Allow Finance group to run app X but not other users" → AppLocker
2. **Quick script/installer control** — simple rules for blocking script types
3. **Supplement WDAC** — use AppLocker for user-targeted rules alongside WDAC for device-wide kernel protection

**AppLocker rule types:**
- **Executable rules** (.exe, .com)
- **Windows Installer rules** (.msi, .msp, .mst)
- **Script rules** (.ps1, .bat, .cmd, .vbs, .js)
- **Packaged app rules** (Microsoft Store apps)
- **DLL rules** (disabled by default — performance impact)

**AppLocker rule conditions:**
- Publisher (certificate-based)
- Path (folder location)
- File hash (exact match)

##### Adaptive Application Controls (Defender for Cloud)

This is a **separate concept** — part of Defender for Servers **Plan 2** only:

- Machine-learning-based recommendations for which apps SHOULD run on a server
- Analyzes running processes → creates recommended allowlist
- Generates alerts when unlisted applications execute
- Can output **WDAC or AppLocker policies** from recommendations
- Works for Azure VMs, Arc-enabled servers, and on-prem servers

**Exam scenario:** "Automatically generate application control policies based on actual server behavior" → **Adaptive Application Controls** (requires Defender for Servers Plan 2)

##### Defender for Cloud Apps — Application Governance

Don't confuse with endpoint app control. **Defender for Cloud Apps** (formerly MCAS) handles **SaaS application control:**

- **Cloud app discovery** — identify shadow IT (unsanctioned SaaS apps)
- **App governance** — monitor OAuth apps registered in Entra ID
  - Detect overprivileged apps, dormant apps, apps accessing sensitive data
  - Set policies to auto-disable suspicious OAuth apps
- **Session controls** (via Conditional Access App Control):
  - Proxy user sessions through Defender for Cloud Apps
  - Block downloads, block copy/paste, apply sensitivity labeling, monitor activity
  - **Exam scenario:** "Prevent users from downloading files from an unmanaged device" → Conditional Access App Control (session policy)
- **Access controls** — block access to apps based on conditions

##### Conditional Access App Control vs. Endpoint App Control

| Aspect | WDAC / AppLocker | Conditional Access App Control |
|--------|-----------------|-------------------------------|
| What it controls | Which executables/scripts run on a device | User sessions within cloud/web apps |
| Where it runs | Endpoint (OS-level) | Cloud proxy (Defender for Cloud Apps) |
| Scope | Servers, workstations | SaaS apps, web apps |
| Example | Block unknown .exe on a server | Block download from SharePoint on unmanaged device |

##### Application Control Decision Tree for SC-100

```
Need to control which EXECUTABLES run on endpoints?
├── YES → Is kernel-mode enforcement needed?
│   ├── YES → WDAC (only option for kernel-mode)
│   └── NO → Do you need per-user rules?
│       ├── YES → AppLocker (or AppLocker supplementing WDAC)
│       └── NO → WDAC (preferred)
└── NO → Need to control SaaS/cloud app access?
    ├── YES → Do you need session-level controls (block downloads, etc.)?
    │   ├── YES → Conditional Access App Control (Defender for Cloud Apps)
    │   └── NO → Conditional Access (block/grant access) + Cloud App Discovery
    └── NO → Need auto-generated allowlists for servers?
        └── YES → Adaptive Application Controls (Defender for Servers Plan 2)
```

##### Exam Scenarios — Application Control

| Scenario | Answer |
|----------|--------|
| "Block unsigned drivers from loading on servers" | WDAC (kernel mode) |
| "Prevent users in HR from running PowerShell scripts" | AppLocker (per-user script rules) |
| "Recommend application allowlists based on server behavior" | Adaptive Application Controls (Plan 2) |
| "Block file downloads from SharePoint on personal devices" | Conditional Access App Control |
| "Most secure application control for Windows endpoints" | WDAC |
| "Deploy application control policy via Intune" | WDAC (custom profile or endpoint security) |
| "Control which apps can run, with minimal policy updates when apps auto-update" | WDAC with FilePublisher rule level |
| "Trust apps deployed by your management tool automatically" | WDAC with managed installer |
| "Identify unsanctioned SaaS apps employees are using" | Defender for Cloud Apps (cloud discovery) |
| "Detect overprivileged OAuth apps in your tenant" | App governance (Defender for Cloud Apps) |
| "Enterprise-managed application control for Windows 11" | WDAC (not Smart App Control) |

##### Study Resources — Application Control
- [WDAC design guide](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/design/wdac-design-guide)
- [WDAC policy rules and file rules](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/design/select-types-of-rules-to-create)
- [WDAC deployment with Intune](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/deployment/deploy-wdac-policies-using-intune)
- [WDAC Wizard tool](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/design/wdac-wizard)
- [AppLocker overview](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/applocker/applocker-overview)
- [Adaptive application controls](https://learn.microsoft.com/en-us/azure/defender-for-cloud/adaptive-application-controls)
- [Defender for Cloud Apps session controls](https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad)
- [App governance in Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/app-governance-manage-app-governance)

##### Self-Test: Application Control
1. Why can't a local admin bypass WDAC but can bypass AppLocker?
2. What's the difference between a WDAC base policy and a supplemental policy?
3. Which WDAC rule level does Microsoft recommend for most organizations?
4. You need to auto-generate application allowlists for servers based on observed behavior. What feature and which Defender plan?
5. A user on a personal (unmanaged) laptop accesses SharePoint. You need to block downloads but allow viewing. What do you use?
6. When would you use AppLocker INSTEAD of (or alongside) WDAC?
7. What is the managed installer trust mechanism in WDAC?
8. How do you deploy a custom WDAC policy to endpoints using Intune?
9. What's the difference between Conditional Access App Control and WDAC?
10. An OAuth app in your tenant has excessive permissions and hasn't been used in 90 days. What detects this?

#### Server Hardening
- **Trusted Launch VMs** — secure boot + vTPM (prevents bootkits/rootkits)
- **Confidential VMs** — DCesv5/ECesv5 series, AMD SEV-SNP, encrypted VM memory
  - Use when: tenant doesn't trust cloud operator, regulatory requirements for data sovereignty
- **Azure Update Manager** — schedule patches, assess compliance, no agent required for Azure VMs
- **Azure Policy guest configuration** — audit OS-level settings inside VMs (password policies, installed software)
- **Privileged Access Workstations (PAWs):**
  - Tier 0: Domain controllers, AD CS, Microsoft Entra Connect
  - Tier 1: Application servers, databases
  - Tier 2: User workstations, devices

### 2. Design Security for Azure Network Infrastructure

#### Azure Firewall — Standard vs. Premium (HIGH-FREQUENCY exam topic)
| Feature | Standard | Premium |
|---------|----------|---------|
| L3-L7 filtering | ✅ | ✅ |
| Threat intelligence (block) | ✅ | ✅ |
| NAT rules | ✅ | ✅ |
| **TLS inspection** | ❌ | **✅** |
| **IDPS (signature-based)** | ❌ | **✅** |
| **URL filtering (full URL path)** | ❌ | **✅** |
| **Web categories (enhanced)** | ❌ | **✅** |

- **Exam tip:** If the question mentions TLS inspection, IDPS, or deep packet inspection → **Premium**
- **Azure Firewall Manager** — centrally manage firewall policies across subscriptions, hub-spoke, and Virtual WAN
- **DNS proxy** — Azure Firewall as DNS proxy for FQDN-based filtering

#### Network Segmentation & Zero Trust Networking
- **Hub-spoke topology** — Azure Firewall in hub for east-west and north-south inspection
- **NSGs** — applied at subnet or NIC level, stateful, 5-tuple rules
- **ASGs** — group VMs logically, reference in NSG rules (no IP management needed)
- **Azure Private Link / Private Endpoints** — access PaaS services over private IP
  - **Exam tip:** Private endpoints + disable public access = Zero Trust for data services
- **Service Endpoints vs. Private Endpoints:**
  - Service endpoints: traffic stays on Azure backbone, source IP is private, destination still public IP
  - Private endpoints: fully private IP in your VNet, DNS resolution required, works cross-region
  - **Private Endpoints are preferred** for Zero Trust

#### VPN & ExpressRoute Security
- **ExpressRoute** encryption options: MACsec (point-to-point at physical layer), IPsec over ExpressRoute private peering
- **ExpressRoute + Private Peering** — for production workloads, doesn't traverse public internet
- **VPN Gateway** — always encrypted (IPsec/IKE), site-to-site, point-to-site, VNet-to-VNet
- **Azure Virtual WAN** — simplified hub-spoke, integrated with Azure Firewall, BGP-based routing

#### DDoS Protection
| Feature | DDoS Network Protection | DDoS IP Protection |
|---------|------------------------|--------------------|
| Scope | All resources in VNet | Single public IP |
| Cost | Monthly + overage | Per-IP pricing |
| WAF discount | ✅ (20%) | ❌ |
| Rapid response team | ✅ | ❌ |
| Cost protection | ✅ | ❌ |

- **Exam tip:** "Protect multiple VNets cost-effectively" → DDoS Network Protection
- **Exam tip:** "Protect a single public IP address" → DDoS IP Protection

### 3. Design Security for Hybrid & Multi-Cloud

#### Azure Arc
- **Arc-enabled servers** — install Connected Machine Agent on non-Azure machines
  - Enables: Azure Policy, Defender for Cloud, Azure Monitor, Update Manager, VM extensions
- **Arc-enabled Kubernetes** — manage any K8s cluster with Azure tools
  - Enables: GitOps (Flux), Azure Policy for K8s, Defender for Containers, monitoring
- **Arc-enabled SQL Server** — extend SQL assessment, Microsoft Entra auth, Defender for SQL
- **Arc-enabled data services** — run Azure SQL MI and PostgreSQL on any infrastructure

#### Multi-Cloud Connectors in Defender for Cloud
- **AWS connector** — uses OIDC federation (no long-lived keys), CloudFormation templates
- **GCP connector** — uses workload identity federation, Terraform/deployment scripts
- Both provide: CSPM, threat protection, regulatory compliance, security recommendations

#### Defender CSPM (Cloud Security Posture Management)
- **Foundational CSPM** (free): Secure score, basic recommendations, Azure Policy
- **Defender CSPM** (paid):
  - **Attack path analysis** — visualize how attacker could move laterally
  - **Cloud security explorer** — query-based resource relationship investigation
  - **Agentless scanning** — VM disks, containers
  - **Data-aware security posture** — discover sensitive data exposure
  - **Governance rules** — assign recommendations to owners with deadlines
  - **External attack surface management (EASM)** integration

### 4. Design Security for Containers

#### Microsoft Defender for Containers
- **Runtime threat detection** — detects suspicious container behavior (crypto mining, shell access, privilege escalation)
- **Vulnerability assessment** — scans images in ACR and running images
- **Kubernetes admission controller** — Azure Policy for AKS enforces pod security standards
- **CI/CD integration** — scan images in pipelines before deployment

#### AKS Security Architecture
- **Private AKS cluster** — API server not exposed to internet, accessed via private endpoint
- **Authorized IP ranges** — restrict who can reach API server (if not fully private)
- **Network policies** — Calico or Azure NPM for pod-to-pod segmentation
- **Workload identity** for pods — replaces deprecated pod-managed identity (aad-pod-identity)
  - Uses OIDC issuer + federated credentials → short-lived tokens from Entra ID
- **Azure CNI** (preferred) vs. **kubenet** — CNI assigns VNet IPs to pods directly
- **Defender profile** on AKS — auto-deploys monitoring agents

### 5. Secrets and Encryption Architecture

#### Azure Key Vault
| Feature | Standard | Premium |
|---------|----------|---------|
| Software-protected keys | ✅ | ✅ |
| HSM-protected keys | ❌ | ✅ (FIPS 140-2 L2) |
| Price | Lower | Higher |

- **Managed HSM** — FIPS 140-2 Level 3, single-tenant, full HSM control, customer-managed root of trust
- **Azure Dedicated HSM** — legacy, Thales Luna HSM, for PKCS#11 / JCE / CNG compliance
- **Exam decision tree:**
  - Need FIPS 140-2 Level 3 + Azure-managed → **Managed HSM**
  - Need FIPS 140-2 Level 3 + full hardware control → **Dedicated HSM**
  - Need HSM-backed keys without single-tenant HSM → **Key Vault Premium**
  - Standard encryption needs → **Key Vault Standard**

#### Encryption Models
- **Server-side encryption (SSE):**
  - Microsoft-managed keys (default for Azure Storage, SQL, etc.)
  - Customer-managed keys (CMK) — stored in Key Vault, you control rotation
  - Customer-provided keys — for Storage Blob only, key passed with each request
- **Client-side encryption** — data encrypted before upload, you manage keys
- **Double encryption** — infrastructure encryption + SSE (Azure Storage, SQL)
- **Azure Disk Encryption (ADE)** — BitLocker (Windows) / DM-Crypt (Linux) on VM disks
- **Encryption at host** — data encrypted before leaving the physical host

### Study Resources — Infrastructure
- [Defender for Servers plans comparison](https://learn.microsoft.com/en-us/azure/defender-for-cloud/plan-defender-for-servers-select-plan)
- [Azure Firewall Premium features](https://learn.microsoft.com/en-us/azure/firewall/premium-features)
- [Private Link and Private Endpoints](https://learn.microsoft.com/en-us/azure/private-link/private-link-overview)
- [Azure Arc overview](https://learn.microsoft.com/en-us/azure/azure-arc/overview)
- [Defender CSPM overview](https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-cloud-security-posture-management)
- [Defender for Containers](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-containers-introduction)
- [AKS security best practices](https://learn.microsoft.com/en-us/azure/aks/operator-best-practices-cluster-security)
- [Azure Key Vault overview](https://learn.microsoft.com/en-us/azure/key-vault/general/overview)
- [WDAC and AppLocker overview](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/wdac-and-applocker-overview)

### Self-Test: Infrastructure (answer these without looking)
1. You need to inspect encrypted east-west traffic between VNets. Which Azure Firewall SKU?
2. An org runs K8s clusters on AWS and Azure. How do you apply uniform security policies?
3. A company needs FIPS 140-2 Level 3 compliance for key storage. What do you recommend?
4. When should you use Private Endpoints vs. Service Endpoints?
5. What is the difference between Defender for Servers Plan 1 and Plan 2?
6. How do you integrate device compliance from Intune into Conditional Access?
7. A company needs to prevent unknown executables from running on servers. WDAC or AppLocker?
8. How does Azure Arc enable multi-cloud security posture management?

---

## 🟡 HIGH — Design Solutions Aligned with Security Best Practices & Priorities (20–25%)

### 1. Zero Trust Architecture (FUNDAMENTAL to entire exam)

Every SC-100 question has Zero Trust thinking baked in. The three principles:
1. **Verify explicitly** — always authenticate and authorize based on all available data points
2. **Use least privilege access** — JIT/JEA, risk-based adaptive policies, data protection
3. **Assume breach** — minimize blast radius, segment access, verify end-to-end encryption, use analytics

#### Zero Trust Pillars & What SC-100 Expects

| Pillar | Key Technologies | What to Recommend |
|--------|-----------------|-------------------|
| **Identity** | Entra ID, Conditional Access, PIM | Risk-based CA + PIM for all admin roles |
| **Endpoints** | Intune, MDE, WDAC | Compliance-based CA + managed device requirement |
| **Data** | Microsoft Purview, sensitivity labels | Auto-labeling + DLP + encryption |
| **Apps** | Defender for Cloud Apps, app proxy | Session controls + CASB discovery |
| **Infrastructure** | Defender for Cloud, Azure Policy | CSPM + security benchmarks |
| **Network** | Azure Firewall, Private Link, NSGs | Microsegmentation + no public endpoints |

### 2. Microsoft Cloud Security Benchmark (MCSB)

- Successor to Azure Security Benchmark (ASB) — now covers multi-cloud
- **12 control families:** Network Security, Identity Management, Privileged Access, Data Protection, Asset Management, Logging & Threat Detection, Incident Response, Posture & Vulnerability Management, Endpoint Security, Backup & Recovery, DevOps Security, Governance & Strategy
- Mapped to: NIST SP 800-53, CIS Controls, PCI-DSS
- **Implemented through:** Azure Policy (built-in initiative: "Microsoft Cloud Security Benchmark")
- **Shows in:** Defender for Cloud regulatory compliance dashboard

### 3. Microsoft Cybersecurity Reference Architectures (MCRA)

- High-level architecture diagrams showing how Microsoft security products fit together
- Study the reference architecture for:
  - **Hybrid identity security** — Entra ID + on-prem AD + Defender for Identity
  - **SIEM + XDR integration** — Sentinel + Defender XDR
  - **Secure remote access** — Entra Private Access + Conditional Access
  - **Multi-cloud security** — Defender for Cloud + Arc

### 4. Ransomware Resiliency Strategy (Microsoft Security Best Practices)

**Three phases of ransomware protection:**

#### Phase 1: Prepare (make your org harder to attack)
- Deploy phishing-resistant MFA (FIDO2, Windows Hello for Business)
- Enable Conditional Access with strong authentication controls
- Implement security baselines and Secure Score improvements
- Train users with attack simulation training

#### Phase 2: Limit (reduce scope of damage)
- **Privileged Access Strategy** — Enterprise Access Model:
  - Control Plane (Entra ID, M365 admin) → highest protection
  - Management Plane (Azure management, Intune) → strong protection
  - Data/Workload Plane (apps, data) → standard protection
- **Lateral movement prevention:**
  - LAPS for local admin passwords
  - Credential Guard (prevents LSASS credential dumping)
  - Disable NTLM where possible, enforce Kerberos with armoring
  - Defender for Identity for on-prem AD attack detection
- **Network segmentation** — microsegmentation, assume breach
- **Privileged Access Workstations (PAWs)** — dedicated devices for admin tasks

#### Phase 3: Respond (recover without paying)
- **Immutable backup vaults** — cannot be deleted even by admins (lock mode)
- **Multi-user authorization (MUA)** — requires 2+ approvers to disable protection
- **Soft delete** — retain deleted backups for 14 days (always on for vaults)
- **Azure Site Recovery** — orchestrated failover to secondary region
- **Isolated recovery environment** — clean room with known-good images
- **Backup strategy:** 3-2-1 (3 copies, 2 media types, 1 offsite) + 1 offline/immutable

#### Key Backup Architecture Decisions
| Scenario | Solution |
|----------|----------|
| Protect backups from ransomware deletion | Immutable vault (locked mode) |
| Require multiple people to modify backup policy | Multi-user authorization (MUA) |
| Cross-region disaster recovery | GRS vault + cross-region restore |
| Recover Azure VMs to different region | Azure Site Recovery |
| Protect on-prem servers to cloud | Azure Backup with MARS agent or MABS |

### 5. Microsoft Cloud Adoption Framework (CAF) — Security Pillar

- **Security methodology:** Define security strategy, plan, ready, adopt, govern, manage
- **CISO workshop:** Understand the modules — especially "Security operations" and "Identity and keys"
- **Landing zone security** — what security controls are in place before workloads deploy
- **Security governance:** Azure Policy (deny, audit, deploy-if-not-exists, modify)

### Study Resources — Best Practices
- [Zero Trust deployment guide](https://learn.microsoft.com/en-us/security/zero-trust/zero-trust-overview)
- [Microsoft Cloud Security Benchmark](https://learn.microsoft.com/en-us/security/benchmark/azure/overview)
- [Microsoft Cybersecurity Reference Architectures](https://learn.microsoft.com/en-us/security/cybersecurity-reference-architecture/mcra)
- [Ransomware protection in Azure](https://learn.microsoft.com/en-us/security/ransomware/)
- [Cloud Adoption Framework — Security](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/secure/)

### Self-Test: Best Practices
1. What are the three Zero Trust principles?
2. What replaced Azure Security Benchmark?
3. Name the three phases of Microsoft's ransomware protection strategy.
4. What is the difference between immutable vaults and soft delete?
5. What is the Enterprise Access Model and its three planes?
6. When would you use MUA for Azure Backup?

---

## 🟡 HIGH — Design Security Operations, Identity, and Compliance Capabilities (25–30%)

### 1. Security Operations Architecture

#### Microsoft Sentinel (Cloud-native SIEM + SOAR)
- **Data connectors** — 200+, including M365, Defender XDR, AWS, GCP, CEF/Syslog, custom (REST API)
- **Analytics rules:**
  - Microsoft security (from Defender products)
  - Scheduled (KQL queries on intervals)
  - Fusion (ML-based multi-stage attack detection)
  - Near-real-time (NRT) — runs every minute
  - Anomaly (ML-based behavioral analytics)
- **Workbooks** — visualization dashboards
- **Playbooks** — Logic Apps for automated response (SOAR)
  - Example: auto-isolate device when high-severity alert fires
- **Watchlists** — import CSV data for enrichment in KQL queries
- **Threat intelligence** — STIX/TAXII feeds, Defender Threat Intelligence integration
- **UEBA (User and Entity Behavior Analytics)** — detect anomalous user/device behavior
- **Content hub** — community + Microsoft-provided solutions (rules, playbooks, workbooks)

#### Microsoft Defender XDR (Extended Detection and Response)
- **Unified incident queue** — correlates alerts across:
  - Defender for Endpoint
  - Defender for Office 365
  - Defender for Identity
  - Defender for Cloud Apps
  - Defender for Cloud
- **Automated investigation and response** — auto-remediates common threats
- **Advanced hunting** — KQL queries across all Defender data
- **Attack disruption** — automatic containment of in-progress attacks (e.g., disable compromised user, isolate device)

#### SIEM + XDR Integration Pattern
- Sentinel is the SIEM — aggregates ALL logs, long-term retention, custom detection
- Defender XDR is the XDR — deep product-specific detection and automated response
- **Bidirectional sync:** Defender XDR incidents appear in Sentinel; Sentinel can trigger playbooks on Defender alerts
- **Exam tip:** "Single pane of glass for all security alerts" → Sentinel (ingests Defender XDR incidents)
- **Exam tip:** "Automated response for endpoint/email threats" → Defender XDR

#### Log Architecture
- **Log Analytics workspace design:**
  - Single workspace for most orgs (simplicity, cross-log correlation)
  - Multiple workspaces for: data residency, billing separation, RBAC needs
- **Data collection rules (DCR)** — filter and route data before ingestion (cost control)
- **Basic logs vs. Analytics logs:**
  - Analytics: full KQL, alerts, 90-day hot retention → higher cost
  - Basic: limited KQL, no alerts, 30-day hot → lower cost
- **Archive tier** — long-term retention (up to 12 years), search jobs to query

### 2. Identity Security Architecture

#### Microsoft Entra ID Protection
- **User risk:** compromised credentials (leaked, anomalous behavior, Tor access)
- **Sign-in risk:** unfamiliar location, anonymous IP, impossible travel, malware-linked IP
- **Risk-based Conditional Access policies (ALL THREE are critical):**
  - High user risk → require password change + MFA
  - High sign-in risk → require MFA
  - Medium sign-in risk → require compliant device or MFA
- **Risk remediation:** self-service password reset, MFA challenge, admin manual intervention

#### Microsoft Entra Privileged Identity Management (PIM)
- **Eligible vs. Active assignments:**
  - Eligible: user must activate (with justification + optional approval)
  - Active: always-on, no activation needed
- **Time-bound access** — define duration (e.g., max 8 hours)
- **Approval workflows** — require specific approvers for sensitive roles
- **Access reviews** — periodic review of role assignments
- **PIM for Groups** — apply PIM to group membership (covers any resource tied to group)
- **PIM for Azure resources** — JIT access to Azure RBAC roles

#### Conditional Access Architecture
- **Named locations** — trusted IP ranges, countries
- **Authentication strengths** — built-in (MFA, passwordless, phishing-resistant) or custom
- **Session controls:**
  - Sign-in frequency (re-authenticate every N hours)
  - Persistent browser session
  - App enforced restrictions
  - Conditional Access App Control (proxy through Defender for Cloud Apps)
- **Token protection** (preview) — binds tokens to device, prevents token theft/replay
- **Continuous Access Evaluation (CAE)** — near-real-time token revocation for critical events

#### Entra Permissions Management
- **Multi-cloud permissions management** — Azure, AWS, GCP
- **Permission Creep Index (PCI)** — identifies over-permissioned identities
- **Right-size permissions** — recommend least-privilege based on actual usage
- **On-demand permissions** — request elevated access with time-bound approval

#### External Identity Management
- **Entra External ID (B2B):** invite external users, they use own IdP, your policies apply
- **Entra External ID (B2C / CIAM):** customer-facing apps, custom sign-up/sign-in flows
- **Cross-tenant access settings:** configure trust relationships, conditional access for B2B
- **Tenant restrictions v2:** prevent data exfiltration via personal accounts

### 3. Compliance Architecture

#### Microsoft Purview Compliance Manager
- **Compliance score** — based on improvement actions across control families
- **Assessments** — mapped to regulatory standards (GDPR, HIPAA, ISO 27001, SOC 2, NIST 800-53)
- **Improvement actions** — Microsoft-managed vs. customer-managed controls
- **Multi-cloud support** — assessments for Azure, AWS, GCP

#### Microsoft Purview Information Protection
- **Sensitivity labels:**
  - Applied to: documents, emails, containers (sites, groups, Teams), schematized data assets
  - Auto-labeling: client-side (Office apps) and service-side (SharePoint, Exchange, OneDrive)
  - Can enforce: encryption, watermarks, headers/footers, access restrictions
- **Label priority** — higher number = more restrictive = higher priority
- **Label scoping** — items (files/emails), containers (groups/sites), schematized data

#### Data Loss Prevention (DLP)
- **DLP policies** — detect and protect sensitive data across:
  - Exchange Online, SharePoint, OneDrive, Teams, Endpoints (Windows/Mac), Power BI, on-prem repositories
- **Sensitive information types (SIT)** — built-in (credit cards, SSN) or custom (regex, keyword dictionaries)
- **Trainable classifiers** — ML-based, pre-trained or custom
- **Endpoint DLP** — monitor/restrict sensitive file actions on devices (copy, print, upload)

#### Data Lifecycle Management & Records Management
- **Retention labels** — keep or delete content after a period
- **Retention policies** — applied to locations (mailbox, site, Teams channel)
- **Records management** — regulatory records, disposition review, file plan
- **Adaptive scopes** — dynamically target policies based on user/group/site attributes

#### Insider Risk Management
- **Insider Risk Management** — detect risky user activities (data theft, policy violations)
- **Communication Compliance** — monitor messages for policy violations
- **Information Barriers** — prevent communication between specific groups (e.g., trading + research)
- **Integration:** Insider Risk signals feed into DLP policies (adaptive protection)

### Study Resources — Security Ops, Identity, Compliance
- [Microsoft Sentinel documentation](https://learn.microsoft.com/en-us/azure/sentinel/overview)
- [Microsoft Defender XDR overview](https://learn.microsoft.com/en-us/microsoft-365/security/defender/microsoft-365-defender)
- [Entra ID Protection](https://learn.microsoft.com/en-us/entra/id-protection/overview-identity-protection)
- [Privileged Identity Management](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure)
- [Microsoft Purview compliance documentation](https://learn.microsoft.com/en-us/purview/purview)
- [Conditional Access overview](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview)

### Self-Test: Security Ops, Identity, & Compliance
1. What's the difference between Sentinel and Defender XDR? When do you use each?
2. What are the three risk-based Conditional Access policies recommended by Microsoft?
3. Explain eligible vs. active assignments in PIM.
4. What is the Permission Creep Index (PCI)?
5. How do sensitivity labels differ from retention labels?
6. When would you use Basic logs vs. Analytics logs in Log Analytics?
7. What is adaptive protection in the context of Insider Risk + DLP?

---

## 🟢 MAINTAIN — Design Security Solutions for Applications and Data (20–25%)

*This was your strongest section. Review to maintain, don't over-invest time here.*

### Quick Review Checklist

#### Application Security
- [ ] DevSecOps: GitHub Advanced Security (CodeQL, secret scanning, Dependabot) + Defender for DevOps
- [ ] WAF: Front Door (global, anycast) vs. Application Gateway (regional, V2)
- [ ] API security: Azure API Management + Defender for APIs + OAuth + mTLS
- [ ] Defender for App Service, Key Vault, Resource Manager, DNS

#### Identity for Applications
- [ ] Managed identity (system-assigned vs. user-assigned) — **always preferred**
- [ ] Workload identity federation — for non-Azure workloads (GitHub Actions, K8s, etc.)
- [ ] Service principals — only when managed identity isn't possible
- [ ] API permissions: Delegated (on behalf of user) vs. Application (daemon/service)
- [ ] Continuous Access Evaluation (CAE) — near-instant token revocation

#### Data Security
- [ ] Microsoft Purview Data Map — discover, classify, govern data across multi-cloud
- [ ] Sensitivity labels on data assets (schematized data labeling)
- [ ] Azure SQL: Always Encrypted (client-side), TDE (server-side), Dynamic Data Masking
- [ ] Azure Storage: encryption at rest (Microsoft-managed or CMK), private endpoints, SAS token best practices
- [ ] Microsoft Defender for Storage — malware scanning, sensitive data detection, anomaly detection
- [ ] Microsoft Defender for SQL — vulnerability assessment, advanced threat protection

### Study Resources — Applications & Data
- [Defender for DevOps](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-devops-introduction)
- [Azure WAF documentation](https://learn.microsoft.com/en-us/azure/web-application-firewall/overview)
- [Managed identities](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview)
- [Microsoft Purview Data Map](https://learn.microsoft.com/en-us/purview/concept-elastic-data-map)

---

## Exam Strategy & Tips

### Time Management
- ~40-60 questions, 120 minutes (some have case studies)
- Don't spend more than 2 minutes per standard question
- Flag and return to uncertain questions — don't get stuck
- Case studies: read the requirements first, THEN the background

### How SC-100 Questions Work
- Most questions are **scenario-based**: "A company has X situation. What should the architect recommend?"
- Look for **keywords** that map to specific solutions:
  - "Minimize cost" → choose the cheaper option (Plan 1 vs. Plan 2, Standard vs. Premium)
  - "Most secure" → choose the strongest option (WDAC over AppLocker, Private Endpoint over Service Endpoint)
  - "Least administrative effort" → choose managed services / automation
  - "Multi-cloud" → Azure Arc + Defender for Cloud
  - "Phishing-resistant" → FIDO2 / Windows Hello / CBA
  - "Prevent token theft" → token protection + CAE
  - "Inspect encrypted traffic" → Azure Firewall Premium
  - "Data residency" → separate Log Analytics workspaces per region

### Common Traps
1. **Don't confuse Defender products** — Defender for Endpoint ≠ Defender for Servers ≠ Defender for Cloud
2. **Service Endpoints vs. Private Endpoints** — Private Endpoints are the Zero Trust answer
3. **Plan 1 vs. Plan 2** — always check if the needed feature is in Plan 1 or Plan 2
4. **Immutable vault vs. soft delete** — immutable prevents modification/deletion; soft delete recovers deleted items
5. **WDAC vs. AppLocker** — WDAC is always preferred unless a specific AppLocker-only scenario is described
6. **Authentication strength** — "phishing-resistant MFA" is a specific Conditional Access authentication strength

---

## Revised 3-Week Intensive Study Plan

### Week 1: Infrastructure Deep Dive (Your Biggest Gap)
- [ ] Day 1-2: Endpoints — MDE, Defender for Servers Plans, WDAC vs. AppLocker
- [ ] Day 3-4: Network — Azure Firewall Standard/Premium, Private Link, NSGs/ASGs, DDoS
- [ ] Day 5: Hybrid — Azure Arc all modalities, multi-cloud connectors
- [ ] Day 6: Containers — AKS security, Defender for Containers, workload identity
- [ ] Day 7: Key Vault/encryption — HSM tiers, CMK, encryption at host, ADE

### Week 2: Best Practices + Security Ops/Identity/Compliance
- [ ] Day 1-2: Zero Trust architecture, MCSB, MCRA
- [ ] Day 3: Ransomware resiliency — all three phases, immutable backups, PAWs
- [ ] Day 4: Sentinel — connectors, analytics rules, playbooks, UEBA, log tiers
- [ ] Day 5: Identity — Entra ID Protection, PIM, Conditional Access patterns
- [ ] Day 6: Compliance — Purview labels, DLP, retention, Insider Risk
- [ ] Day 7: Review weak areas from Week 1, take practice questions

### Week 3: Integration + Practice Exams
- [ ] Day 1: Application & data security review (your strongest area — quick pass)
- [ ] Day 2-3: Practice exam #1 + review every wrong answer with docs
- [ ] Day 4-5: Practice exam #2 + targeted study on missed topics
- [ ] Day 6: Quick-reference table review, self-test questions from this guide
- [ ] Day 7: Light review only — rest before exam

### Recommended Practice Exams
- [Microsoft Learn Practice Assessment (free)](https://learn.microsoft.com/en-us/credentials/certifications/cybersecurity-architect-expert/practice/assessment?assessment-type=practice&assessmentId=79)
- [MeasureUp SC-100 Practice Exam (paid, most similar to real exam)](https://www.measureup.com/)
- [Whizlabs SC-100 Practice Tests](https://www.whizlabs.com/)

---

## Quick-Reference: Key Decision Matrix for SC-100

| Scenario | Solution |
|----------|----------|
| Protect VMs from ransomware backup deletion | Immutable Azure Backup vault (locked) + MUA |
| Secure admin access to Azure portal | PIM (eligible assignments) + Conditional Access + PAW |
| Multi-cloud security posture visibility | Defender for Cloud + Defender CSPM + Azure Arc |
| Protect web apps from OWASP Top 10 | Azure WAF (Front Door for global, App GW for regional) |
| Scan containers for vulnerabilities | Defender for Containers |
| Restrict which apps can run on endpoints | WDAC (preferred over AppLocker) |
| Secure east-west traffic in Azure | Azure Firewall (hub-spoke) + NSGs + microsegmentation |
| Detect lateral movement in on-prem AD | Microsoft Defender for Identity |
| Phishing-resistant MFA | FIDO2 / Windows Hello for Business / Certificate-based auth |
| Secure secrets for applications | Azure Key Vault + managed identity |
| Protect against DDoS | Azure DDoS Protection (Network or IP) + WAF |
| Multi-cloud server management | Azure Arc-enabled servers + Defender for Cloud |
| Inspect encrypted network traffic | Azure Firewall **Premium** (TLS inspection) |
| Detect anomalous user behavior | UEBA in Microsoft Sentinel |
| Single pane for all security alerts | Microsoft Sentinel (ingests Defender XDR) |
| Prevent data exfiltration via personal accounts | Tenant restrictions v2 |
| Over-permissioned cloud identities | Entra Permissions Management (PCI) |
| FIPS 140-2 Level 3 key protection | Managed HSM |
| Secure non-Azure workload identity | Workload identity federation |
| Auto-respond to high-severity alerts | Sentinel playbooks (Logic Apps) |
| Classify and label sensitive data at scale | Purview sensitivity labels + auto-labeling policies |
| Prevent sensitive data sharing | Microsoft Purview DLP |

---

*You scored 623 — you're 77 points away. Infrastructure is where the biggest gains are. Focus 50% of your time there, 30% on best practices + secops/identity/compliance, and 20% maintaining your app/data knowledge. You've got this!*
