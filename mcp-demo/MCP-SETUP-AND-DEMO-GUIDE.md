# Microsoft Learn MCP Server — Setup and Demo Guide

This guide walks through setting up Microsoft MCP (Model Context Protocol) servers in VS Code for querying official documentation, Azure resources, and Microsoft Entra ID Conditional Access policies — then demonstrates real-world usage against a live tenant.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step 1 — Configure MCP Servers](#step-1--configure-mcp-servers)
3. [Step 2 — Add Copilot Instructions](#step-2--add-copilot-instructions)
4. [Step 3 — Verify Server Connectivity](#step-3--verify-server-connectivity)
5. [Demo 1 — Search Official Documentation](#demo-1--search-official-documentation)
6. [Demo 2 — Fetch Full Documentation Pages](#demo-2--fetch-full-documentation-pages)
7. [Demo 3 — List Conditional Access Policies via Microsoft Graph](#demo-3--list-conditional-access-policies-via-microsoft-graph)
8. [Demo 4 — Compare Tenant to Microsoft Recommendations](#demo-4--compare-tenant-to-microsoft-recommendations)
9. [MCP Server Reference](#mcp-server-reference)
10. [Appendix — Conditional Access Deployment Guidance](#appendix--conditional-access-deployment-guidance)

---

## Prerequisites

- **Visual Studio Code** ([install](https://code.visualstudio.com/docs/setup/setup-overview))
- **GitHub Copilot** extension with an active subscription ([setup](https://code.visualstudio.com/docs/copilot/setup))
- An Azure / Entra ID account with access to your target tenant
- No additional authentication is required for the Microsoft Learn MCP server

---

## Step 1 — Configure MCP Servers

Create `.vscode/mcp.json` in your workspace root. This registers three MCP servers:

| Server | Endpoint | Purpose |
|---|---|---|
| `microsoft-learn` | `https://learn.microsoft.com/api/mcp` | Search, fetch, and browse official Microsoft documentation and code samples |
| `azure` | `https://mcp.azure.com/sse` | Query Azure subscriptions, resource groups, and resources via Azure Resource Graph |
| `microsoft-graph` | `https://mcp.graph.microsoft.com/sse` | Query Entra ID objects: Conditional Access policies, users, groups, service principals |

```json
{
  "servers": {
    "microsoft-learn": {
      "type": "http",
      "url": "https://learn.microsoft.com/api/mcp"
    },
    "azure": {
      "type": "http",
      "url": "https://mcp.azure.com/sse",
      "headers": {
        "x-tenant-id": "${input:azure-tenant-id}"
      }
    },
    "microsoft-graph": {
      "type": "http",
      "url": "https://mcp.graph.microsoft.com/sse",
      "headers": {
        "x-tenant-id": "${input:graph-tenant-id}"
      }
    }
  },
  "inputs": [
    {
      "id": "azure-tenant-id",
      "type": "promptString",
      "description": "Azure Tenant ID",
      "default": "b22dee98-83da-4207-b9ab-5ba931866f44"
    },
    {
      "id": "graph-tenant-id",
      "type": "promptString",
      "description": "Microsoft Graph Tenant ID",
      "default": "b22dee98-83da-4207-b9ab-5ba931866f44"
    }
  ]
}
```

> **Note:** Replace the default tenant ID with your own. VS Code will prompt for confirmation on first use. The `microsoft-learn` server requires no authentication. The `azure` and `microsoft-graph` servers authenticate via your signed-in Azure identity.

---

## Step 2 — Add Copilot Instructions

Create `.github/copilot-instructions.md` so the agent knows when and how to use the MCP tools:

```markdown
---
applyTo: '**'
---
## Target Tenant

The default Azure / Entra ID tenant for all queries is **b22dee98-83da-4207-b9ab-5ba931866f44**.

## Querying Microsoft Documentation

You have access to MCP tools called `microsoft_docs_search`, `microsoft_docs_fetch`,
and `microsoft_code_sample_search` - these tools allow you to search through and fetch
Microsoft's latest official documentation and code samples, and that information might
be more detailed or newer than what's in your training data set.

When handling questions around how to work with native Microsoft technologies, such as
C#, F#, ASP.NET Core, Microsoft.Extensions, NuGet, Entity Framework, the `dotnet`
runtime - please use these tools for research purposes when dealing with specific /
narrowly defined questions that may occur.

## Querying Microsoft Graph (Entra ID / Conditional Access)

You have access to MCP tools prefixed with `microsoft_graph_` — use these to query the
Microsoft Graph API for Entra ID objects including Conditional Access policies, users,
groups, service principals, and directory roles.

Key queries for Conditional Access policy reporting:
- **List all CA policies:** `GET /beta/identity/conditionalAccess/policies?$count=true&$select=id,displayName,createdDateTime,modifiedDateTime,state&$orderBy=modifiedDateTime desc`
- **Enabled policies only:** add `$filter=state eq 'enabled'`
- **Policy details:** `GET /v1.0/identity/conditionalAccess/policies/{id}` for full grant controls, session controls, and conditions
- Use `microsoft_graph_suggest_queries` to discover the right Graph endpoint for any Entra ID question.
- Use `microsoft_graph_get` to execute Graph API calls.
- Use `microsoft_graph_list_properties` to explore response schemas.

## Querying Azure Resources

You have access to Azure MCP tools — use these for querying Azure subscriptions,
resource groups, and resources via Azure Resource Graph. These tools can help correlate
Azure resource configurations with Conditional Access policy assignments.
```

---

## Step 3 — Verify Server Connectivity

1. Open VS Code and navigate to your workspace
2. Open the **Command Palette** (`Ctrl+Shift+P`)
3. Run **MCP: List Servers**
4. Confirm all three servers appear: `microsoft-learn`, `azure`, `microsoft-graph`
5. Start each server — VS Code will prompt for tenant ID on Azure/Graph servers

---

## Demo 1 — Search Official Documentation

**Prompt (Agent mode):**
> Use `microsoft_docs_search` to find the current guidance for configuring Microsoft Entra ID Conditional Access in a GCC High environment. Summarize key requirements.

**What happens:** The agent calls the Learn MCP Server's search tool, retrieves relevant documentation chunks, and synthesizes a summary covering:

- **Licensing:** Entra ID P1 minimum (P2 for risk-based policies)
- **Endpoints:** GCC High uses `https://login.microsoftonline.us` (not the commercial endpoint)
- **Feature availability:** Conditional Access, MFA, PIM, and most ID Protection detections are available in GCC High
- **Gaps:** Microsoft Entra threat intelligence risk detection and My Staff portal are not available
- **B2B:** Only supported between organizations both in the Azure US Government cloud

### Key GCC High Endpoints

| Service | GCC High Endpoint |
|---|---|
| Portal | `https://portal.azure.us` |
| Authentication / Authority | `https://login.microsoftonline.us` |
| Microsoft Graph API | `https://graph.microsoft.us` |

---

## Demo 2 — Fetch Full Documentation Pages

**Prompt (Agent mode):**
> Use `microsoft_docs_fetch` to retrieve the official GCC High Conditional Access documentation in markdown format.

**What happens:** The agent fetches the complete page content from two key documents:

1. **[Plan a Conditional Access deployment](https://learn.microsoft.com/entra/identity/conditional-access/plan-conditional-access)** — full deployment guide with phased rollout, naming standards, and best practices
2. **[Microsoft Entra feature availability](https://learn.microsoft.com/entra/identity/authentication/feature-availability)** — complete feature parity matrix for US Government clouds

The agent returns clean markdown with tables, headers, and links — ready for offline reference or inclusion in reports.

---

## Demo 3 — List Conditional Access Policies via Microsoft Graph

**Prompt (Agent mode):**
> List all Conditional Access policies in this tenant

**What happens:** The agent:
1. Calls `microsoft_graph_suggest_queries` to find the right Graph endpoint
2. Executes `microsoft_graph_get` with:
   ```
   GET /beta/identity/conditionalAccess/policies?$count=true
     &$select=id,displayName,createdDateTime,modifiedDateTime,state
     &$orderBy=modifiedDateTime desc&$top=30
   ```
3. Returns a formatted table of all policies

### Example Output (21 policies found)

#### Enabled Policies

| Policy | Created | Last Modified |
|---|---|---|
| CA001 - Block legacy authentication | 2026-01-29 | 2026-01-29 |
| CA002 - Securing security info registration | 2026-01-29 | 2026-01-29 |
| CA003 - Require Passwordless for ADMIN roles | 2026-01-29 | 2026-01-30 |
| CA004 - Require MFA for all users | 2026-01-29 | 2026-02-03 |
| CA005 - Require multifactor authentication for guest access | 2026-01-29 | 2026-01-29 |
| CA006 - Require Passwordless for Azure management | 2026-01-29 | 2026-01-30 |
| CA007 - Require Passwordless for risky sign-ins | 2026-01-29 | 2026-01-30 |
| CA008 - Require Passwordless for high-risk users | 2026-01-29 | 2026-01-30 |
| CA009 - Require MFA to register or join devices | 2026-01-29 | 2026-01-30 |
| CA010 - Workload Protection | 2026-02-13 | 2026-02-13 |
| CA012 - Workload Risk Protection | 2026-02-13 | — |
| Multifactor authentication for Microsoft partners and vendors | 2026-01-28 | 2026-02-06 |
| Reauthentication on signin risk for Microsoft partners and vendors | 2026-01-28 | 2026-02-06 |

#### Disabled Policies

| Policy | Last Modified |
|---|---|
| CA012 - Block all high risk agents from accessing all resources | 2026-02-28 |
| CA013 - Block access to Office Apps for users with Insider Risk (Preview) | 2026-02-28 |
| CA014 - Require Compliant Device | 2026-02-18 |
| Block device code flow | 2026-02-28 |
| Require Microsoft-managed remediation for high-risk local user | 2026-02-28 |
| Merged MFA for Microsoft partners and vendors and CA004 | 2026-02-28 |
| Secure password change on high user risk for Microsoft partners and vendors | 2026-02-28 |
| Security info registration for Microsoft partners and vendors | 2026-02-13 |

---

## Demo 4 — Compare Tenant to Microsoft Recommendations

**Prompt (Agent mode):**
> What does MCP Learn recommend for deployment?

**What happens:** The agent combines data from both MCP servers:
1. Searches Learn for the official deployment guidance
2. Cross-references your live tenant policies from Graph
3. Produces a gap analysis

### Microsoft's Recommended 3-Phase Deployment

| Phase | Timeframe | Policies |
|---|---|---|
| **1: Foundation** | Week 1–2 | Block legacy auth, Secure MFA registration, Phishing-resistant MFA for admins |
| **2: Core Auth** | Week 2–3 | MFA for all users & guests, Approved apps/app protection on mobile, MFA for device join |
| **3: Advanced** | Week 3–4 | Risk-based sign-in/user policies (P2), Token protection, Block device code flow, Block auth transfer, PAW policies |

### Gap Analysis Against Your Tenant

| Recommendation | Status |
|---|---|
| Block legacy auth | ✅ CA001 enabled |
| Secure MFA registration | ✅ CA002 enabled |
| Phishing-resistant for admins | ✅ CA003 enabled |
| MFA for all users | ✅ CA004 enabled |
| MFA for guests | ✅ CA005 enabled |
| Passwordless for Azure mgmt | ✅ CA006 enabled |
| Risk-based sign-in | ✅ CA007 enabled |
| Risk-based user | ✅ CA008 enabled |
| MFA for device registration | ✅ CA009 enabled |
| Block device code flow | ⚠️ Disabled |
| Block auth transfer | ❌ Not present |
| Token protection | ❌ Not present |
| Require compliant device | ⚠️ CA014 disabled |
| PAW for privileged users | ❌ Not present |

**Summary:** Phase 1 & 2 are fully covered. Phase 3 advanced protections have gaps — consider enabling device code flow block and adding token protection, authentication transfer block, and PAW policies.

### Key Best Practices from Microsoft Learn

- **Report-only mode first** — run each new policy in report-only for at least 1 week
- **Always exclude break-glass accounts** from all policies
- **Target "All resources"** rather than individual apps
- **Minimize policy count** — 195 policy limit per tenant
- **Use naming standards** — e.g., `CA001 - Block legacy authentication`
- **Plan contingency policies** — disabled emergency policies (e.g., `EM01 - ENABLE IN EMERGENCY`)
- **Block unexpected countries** via named locations

---

## MCP Server Reference

### Tools Available per Server

| Server | Tools | Auth Required |
|---|---|---|
| **microsoft-learn** | `microsoft_docs_search`, `microsoft_docs_fetch`, `microsoft_code_sample_search` | No |
| **azure** | Azure Resource Graph queries, subscription/resource group listing | Yes (Azure identity) |
| **microsoft-graph** | `microsoft_graph_get`, `microsoft_graph_suggest_queries`, `microsoft_graph_list_properties` | Yes (Azure identity) |

### Useful Graph Queries for Conditional Access

```
# List all CA policies (sorted by last modified)
GET /beta/identity/conditionalAccess/policies?$count=true
  &$select=id,displayName,createdDateTime,modifiedDateTime,state
  &$orderBy=modifiedDateTime desc

# Enabled policies only
GET /beta/identity/conditionalAccess/policies?$count=true
  &$filter=state eq 'enabled'
  &$select=id,displayName,createdDateTime,modifiedDateTime,state,grantControls
  &$orderBy=modifiedDateTime desc

# Search by name
GET /beta/identity/conditionalAccess/policies?$count=true
  &$filter=contains(tolower(displayName), tolower('MFA'))
  &$select=id,displayName,modifiedDateTime

# Full policy details
GET /v1.0/identity/conditionalAccess/policies/{policy-id}
```

---

## Appendix — Conditional Access Deployment Guidance

### GCC High Environment Notes

| Consideration | Detail |
|---|---|
| Authentication endpoint | `https://login.microsoftonline.us` |
| Graph API | `https://graph.microsoft.us` |
| Portal | `https://portal.azure.us` |
| B2B collaboration | Only between Azure US Government orgs |
| Feature gap | Microsoft Entra threat intelligence detection unavailable |
| Feature gap | My Staff portal unavailable |

### Conditional Access Optimization Agent (Preview)

Microsoft's Copilot-powered agent can automate safe CA policy deployment:
- Targets report-only policies scoped to all users
- Creates a 5-phase rollout (low-impact → high-impact groups)
- Auto-pauses if >10% of sign-ins are blocked
- Requires Entra ID P1 + Security Copilot compute units

---

## Resources

- [Microsoft Learn MCP Server overview](https://learn.microsoft.com/en-us/training/support/mcp)
- [Get Started with Learn MCP in VS Code](https://learn.microsoft.com/en-us/training/support/mcp-get-started)
- [Plan a Conditional Access deployment](https://learn.microsoft.com/entra/identity/conditional-access/plan-conditional-access)
- [Microsoft Entra feature availability (US Gov)](https://learn.microsoft.com/entra/identity/authentication/feature-availability)
- [FedRAMP High — ID & auth controls](https://learn.microsoft.com/entra/standards/fedramp-identification-and-authentication-controls)
- [National clouds — authentication endpoints](https://learn.microsoft.com/entra/identity-platform/authentication-national-cloud)
