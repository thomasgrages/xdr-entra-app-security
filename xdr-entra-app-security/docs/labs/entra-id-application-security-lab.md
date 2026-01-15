
# Entra ID Workload Identity Security Lab  
## Detecting and Investigating Application & Service Principal Compromise (Defender‑Assisted)

> **Purpose**  
This lab provides a prescriptive, end‑to‑end walkthrough for securing Microsoft Entra ID applications and service principals, **detecting application compromise**, and **investigating malicious workload identity activity** using Microsoft Defender–integrated telemetry (Entra ID, Defender for Cloud Apps, Microsoft Graph, and audit logs).  
The focus is on **application and service principal compromise**, with Defender used for detection and investigation — not full cross‑endpoint XDR correlation.

---

## 0. Prerequisites (Exact)

### Licensing
- Microsoft Entra ID (P1 minimum, **P2 recommended**)
- Microsoft Defender for Cloud Apps
- Microsoft Defender (M365 security portal access for audit, alerts, hunting)

### Required Roles
- **Cloud Application Administrator**
- **Global Reader**
- **Security Reader** or **Security Operator**
- Azure RBAC role for managed identity testing (e.g., **Storage Blob Data Reader**)

### Portals
- Entra Admin Center: https://entra.microsoft.com
- Microsoft 365 Defender portal: https://security.microsoft.com
- Defender for Cloud Apps: https://portal.cloudappsecurity.com

---

## 1. Threat Model – How Applications Get Compromised

| Attack Vector | Description |
|--------------|-------------|
| Credential theft | Secrets or certificates stolen from pipelines, repos, or disk |
| Permission abuse | Over‑privileged Graph or M365 permissions |
| Persistence | Attacker adds new credentials to existing app |
| Supply chain | Compromised multi‑tenant SaaS app |
| Token abuse | OAuth access tokens reused from abnormal location |

> **Key insight:** Service principals authenticate non‑interactively — attackers love them because MFA, device state, and user behavior analytics are bypassed.

---

## 2. Secure Configuration (Foundation)

*(Keep these steps — they directly reduce blast radius and false positives during detection.)*

### 2.1 Prefer Managed Identities
- Use **system‑assigned managed identities** where possible
- Assign **minimal Azure RBAC**
- No secrets, no certificates, no rotation risk

---

### 2.2 Least‑Privilege App Registration

1. Entra → **App registrations** → **New registration**
   - Name: `Lab-LeastPrivilege-App`
   - Single tenant
2. **API permissions**
   - Microsoft Graph → **Application permissions**
   - Add only what is required (example: `User.Read.All`)
3. Grant admin consent

🚫 Avoid:
- `Directory.ReadWrite.All`
- `Sites.ReadWrite.All`

---

### 2.3 Ownership & Governance

- Remove **direct owners** from high‑priv apps
- Manage apps using **Entra ID roles (PIM)**
- Recommended role: `Cloud Application Administrator` (eligible)

---

### 2.4 Credential Hygiene

**Preferred**
- Certificates over secrets

**If secrets**
- Expiry: **≤ 90 days**
- Store in **Azure Key Vault**
- Rotate aggressively

---

## 3. Detection – Identifying App Compromise (Defender Coverage)

### 3.1 OAuth App Detections (Defender for Cloud Apps)

Navigate: **Control → Policies → OAuth app**

Create / enable:

| Policy | Signal |
|------|-------|
| OAuth app – High permissions granted | Privilege escalation |
| OAuth app – New admin consent | Unauthorized permission grant |
| OAuth app – Credentials added | Persistence |
| OAuth app – Unknown publisher | Supply‑chain risk |
| OAuth app – Mass user consent | Abuse or phishing |

Severity guidance:
- High permissions / credential addition → **High**
- Unknown publisher / mass consent → **Medium**

---

### 3.2 Entra ID Audit Log Detections (High‑Confidence)

Monitor these events:

- **Add service principal credentials**
- **Update application – Certificates and secrets**
- **Grant admin consent to application**
- **Add application**
- **Add service principal**

These are **near‑certain indicators** of attacker persistence when unexpected.

---

## 4. Investigation – Defender‑Led App Compromise Workflow

> This section mirrors real incident response: **alert → validation → scope → impact → response**

---

### 4.1 Validate Service Principal Sign‑ins

```kusto
SigninLogs
| where SignInType == "ServicePrincipal"
| project TimeGenerated, AppDisplayName, AppId, IPAddress, Location
| order by TimeGenerated desc
