
# Defender XDR Lab: Securing, Detecting & Investigating Entra ID Applications

## Lab Objective
This lab teaches how to secure Entra ID applications, detect their compromise, and investigate malicious app activity using Defender XDR.

---

## Section 1 – Understanding the App Threat Model

### Key Objects
| Object | Description |
|------|-------------|
| Application | Blueprint |
| Service Principal | Runtime identity |
| Credential | Primary attacker target |
| Permissions | Blast radius |

---

## Section 2 – Secure Entra ID Applications

### 2.1 Use Managed Identities
- Replace secrets and certificates where possible
- Validate no credentials exist

### 2.2 Apply Least Privilege
- Minimize Graph permissions
- Prefer resource-specific consent

### 2.3 Ownership Model
- Remove direct owners from high-priv apps
- Govern via Entra ID roles

### 2.4 Credential Hardening
- Certificates over secrets
- Rotate credentials
- Disable unused credentials

### 2.5 Remove Unused Apps
- Identify stale service principals
- Remove abandoned multitenant apps

---

## Section 3 – Detect App Compromise

### Key Signals
- New application creation
- Credential additions
- Unexpected admin consent
- New resource access
- ASN / geo anomalies

> Defender for Cloud Apps and Entra audit logs provide most of this telemetry.

---

## Section 4 – Investigate Using Defender XDR

### 4.1 Service Principal Sign-ins
- Non-interactive sign-ins
- IP, ASN, location review

### 4.2 Audit Logs
- Credential additions
- Permission changes
- Persistence creation

### 4.3 Microsoft Graph Usage
- Directory enumeration
- Mailbox or site access

### 4.4 Unified Audit Log
- SharePoint access
- Exchange mailbox reads

---

## Section 5 – Incident Response Actions

1. Disable service principal
2. Remove credentials
3. Revoke permissions
4. Assess data exposure
5. Destroy attacker persistence

---

## Outcome
Security teams gain visibility, control, and confidence in defending Entra ID application identities.
