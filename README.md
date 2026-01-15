# Entra ID Workload Identity Application Security

Hands-on lab and investigation guide for **securing, detecting, and investigating compromise of Microsoft Entra ID applications and service principals**.

This repository focuses on **workload identity security** — an often overlooked attack surface — and demonstrates how attackers abuse application credentials, permissions, and OAuth consent, along with how defenders can detect and investigate that activity using **Defender‑integrated identity telemetry**.

---

## What This Repository Covers

✅ Secure configuration of Entra ID applications and service principals  
✅ Least‑privilege permission design for Microsoft Graph  
✅ Managed identities and credential hygiene  
✅ Detection of application compromise and persistence  
✅ OAuth abuse and admin consent monitoring  
✅ Investigation using Entra ID audit logs, sign‑in telemetry, Microsoft Graph, and Unified Audit Log  
✅ Defender‑assisted investigation workflows (without requiring full XDR)

---

## Why This Matters

Most organizations invest heavily in protecting **users**, but significantly under‑secure **workload identities**.

Applications and service principals typically:
- Have long‑lived credentials
- Bypass MFA and device controls
- Are heavily over‑privileged
- Are rarely monitored with the same rigor as users

As a result, they are frequently abused for:
- Persistence
- Data access and exfiltration
- Supply‑chain compromise
- Privilege escalation

This repo exists to close that gap.

---

## Repository Structure

```text
docs/
├── labs/
│   └── entra-id-workload-identity-app-compromise-lab.md
│
├── detections/
│   └── kql/
│       ├── 4-2-credential-additions.kql
│       ├── 4-3-graph-application-activity.kql
│       └── 4-4-unified-audit-app-access.kql
│
└── mitre/
    ├── entra-id-applications-mitre-mapping.md
    └── entra-id-applications-navigator-layer.json
