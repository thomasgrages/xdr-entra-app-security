# MCP Server Lab Guide — Setup, Usage, Optimization, and Defender XDR

> **Audience:** Security professionals, IT admins, and developers exploring AI-assisted workflows  
> **Duration:** ~90 minutes  
> **Prerequisites:** VS Code with GitHub Copilot (Chat agent mode), Node.js 18+, npm, a Microsoft 365/Azure tenant (for Defender labs)

---

## Table of Contents

1. [Part 1 — What Is MCP?](#part-1--what-is-mcp)
2. [Part 2 — Setting Up MCP Servers](#part-2--setting-up-mcp-servers)
3. [Part 3 — Using MCP Tools in Practice](#part-3--using-mcp-tools-in-practice)
4. [Part 4 — Optimizing with Context Mode](#part-4--optimizing-with-context-mode)
5. [Part 5 — Microsoft Defender XDR Integration](#part-5--microsoft-defender-xdr-integration)
6. [Part 6 — Incident Response Walkthrough](#part-6--incident-response-walkthrough)
7. [Appendix — Troubleshooting](#appendix--troubleshooting)

---

## Part 1 — What Is MCP?

### Overview

**Model Context Protocol (MCP)** is an open standard that allows AI models (like GitHub Copilot) to connect to external tools and data sources. Instead of the AI only knowing what's in its training data, MCP lets it:

- **Search live documentation** (Microsoft Learn)
- **Automate browsers** (Playwright)
- **Query library APIs** (Context7)
- **Take security response actions** (Defender XDR)

### How It Works

```
┌─────────────────┐     MCP Protocol      ┌──────────────────┐
│                  │  ←───────────────────→ │  MCP Server 1    │
│   AI Agent       │                        │  (Microsoft Learn)│
│   (Copilot)      │  ←───────────────────→ │  MCP Server 2    │
│                  │                        │  (Playwright)     │
│                  │  ←───────────────────→ │  MCP Server 3    │
│                  │                        │  (Defender XDR)   │
└─────────────────┘                        └──────────────────┘
```

- **stdio servers** run as local processes (e.g., Playwright, Context Mode)
- **HTTP servers** connect to remote APIs (e.g., Microsoft Learn, Context7)
- The AI agent decides which tool to call based on your prompt

### Key Concepts

| Term | Definition |
|------|-----------|
| **MCP Server** | A process or endpoint that exposes tools to the AI |
| **Tool** | A specific function the server provides (e.g., `microsoft_docs_search`) |
| **stdio** | Server runs as a local child process, communicating via stdin/stdout |
| **HTTP** | Server is a remote API the AI connects to over HTTPS |
| **Context Window** | The limited "memory" the AI has during a conversation |

---

## Part 2 — Setting Up MCP Servers

### Lab 2.1 — Create the Configuration File

All MCP servers are configured in `.vscode/mcp.json` in your project root.

**Step 1:** Create the file:

```
📁 your-project/
├── 📁 .vscode/
│   └── 📄 mcp.json          ← MCP server config
└── ...
```

**Step 2:** Start with a minimal config — just Microsoft Learn:

```json
{
  "servers": {
    "MicrosoftLearn": {
      "type": "http",
      "url": "https://learn.microsoft.com/api/mcp"
    }
  }
}
```

**Step 3:** Restart VS Code. Open Copilot Chat in **Agent mode** (not Ask or Edit). You should see MicrosoftLearn listed in the available tools.

> **Checkpoint:** Ask Copilot: *"Search Microsoft Learn for Conditional Access overview"* — it should call the `microsoft_docs_search` tool.

---

### Lab 2.2 — Add Playwright (Browser Automation)

Add the Playwright server to your `mcp.json`:

```json
{
  "servers": {
    "MicrosoftLearn": {
      "type": "http",
      "url": "https://learn.microsoft.com/api/mcp"
    },
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"],
      "type": "stdio"
    }
  }
}
```

No install required — `npx` downloads it on first use. Restart VS Code.

> **Checkpoint:** Ask Copilot: *"Navigate to https://learn.microsoft.com and take a snapshot"* — it should launch a browser and return page content.

---

### Lab 2.3 — Add Context7 (Library Documentation)

Context7 provides up-to-date docs for any library — useful when the AI's training data is outdated.

```json
{
  "servers": {
    "MicrosoftLearn": {
      "type": "http",
      "url": "https://learn.microsoft.com/api/mcp"
    },
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"],
      "type": "stdio"
    },
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "${input:context7ApiKey}"
      }
    }
  },
  "inputs": [
    {
      "id": "context7ApiKey",
      "type": "promptString",
      "description": "Context7 API Key (get one at https://context7.com)",
      "password": true
    }
  ]
}
```

**Note the `inputs` section** — this prompts for the API key securely at runtime instead of hardcoding it.

> **Checkpoint:** Ask Copilot: *"Look up the latest React useEffect documentation using Context7"*

---

### Lab 2.4 — Understanding Server Types

| Server | Type | Runs Where | Auth | Use Case |
|--------|------|-----------|------|----------|
| Microsoft Learn | HTTP | Remote (Microsoft) | None | Search/fetch official docs |
| Playwright | stdio | Local process | None | Browser automation, screenshots |
| Context7 | HTTP | Remote (Context7) | API key | Library docs and code examples |

**Discussion Point:** When would you choose stdio vs HTTP? (Answer: stdio for tools that need local system access like browsers/filesystems; HTTP for cloud APIs.)

---

## Part 3 — Using MCP Tools in Practice

### Lab 3.1 — Searching Documentation

Open Copilot Chat (Agent mode) and try these prompts:

**Prompt 1 — Broad search:**
> "Search Microsoft Learn for Zero Trust architecture principles"

Watch the tool call — Copilot invokes `microsoft_docs_search` and returns summarized results.

**Prompt 2 — Fetch a specific page:**
> "Fetch the full content of the Microsoft Learn page on Conditional Access policies"

This uses `microsoft_docs_fetch` to return the complete page in markdown.

**Prompt 3 — Find code samples:**
> "Find code samples for Microsoft Graph API authentication in Python"

Uses `microsoft_code_sample_search` with language filtering.

---

### Lab 3.2 — Browser Automation with Playwright

**Prompt 1 — Navigate and snapshot:**
> "Open https://security.microsoft.com and take a snapshot of the page"

**Prompt 2 — Interact with a page:**
> "Navigate to https://learn.microsoft.com, search for 'Defender XDR', and snapshot the results"

**Key Observation:** Notice the snapshot output size. A single Playwright snapshot can be **56 KB** — this matters for context optimization (Part 4).

---

### Lab 3.3 — Combining Multiple Servers

The real power is chaining tools. Try:

> "Search Microsoft Learn for Defender for Endpoint onboarding methods, then look up the latest Microsoft Graph security API docs using Context7"

Copilot will call multiple MCP servers in one response.

---

## Part 4 — Optimizing with Context Mode

### The Problem

Every MCP tool call returns data into the AI's **context window** — which is limited. Over a long session:

| Tool Call | Typical Output Size |
|-----------|-------------------|
| Playwright snapshot | ~56 KB |
| 20 GitHub issues | ~59 KB |
| Access log file | ~45 KB |
| Microsoft Learn page | ~15-30 KB |

After 30 minutes, **40%+ of your context can be consumed by raw tool output**. When context fills up, the AI "compacts" (forgets older messages) and loses track of what you were doing.

### Lab 4.1 — Install Context Mode

**Step 1:** Install globally:

```bash
npm install -g context-mode
```

**Step 2:** Add to `.vscode/mcp.json`:

```json
"context-mode": {
  "command": "context-mode",
  "type": "stdio"
}
```

**Step 3:** Create hooks at `.github/hooks/context-mode.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      { "type": "command", "command": "context-mode hook vscode-copilot pretooluse" }
    ],
    "PostToolUse": [
      { "type": "command", "command": "context-mode hook vscode-copilot posttooluse" }
    ],
    "SessionStart": [
      { "type": "command", "command": "context-mode hook vscode-copilot sessionstart" }
    ]
  }
}
```

**Step 4:** Restart VS Code.

---

### Lab 4.2 — See the Savings

After some tool calls, ask:

> "Show me context mode stats"

This invokes `ctx_stats` and shows you:
- Per-tool breakdown of context savings
- Total tokens consumed vs saved
- Savings ratio (target: ~98% with hooks)

### How It Works

```
Without Context Mode:
  Playwright snapshot (56 KB) → ALL 56 KB enters context window

With Context Mode:
  Playwright snapshot (56 KB) → Indexed into SQLite → 299 bytes enters context
  
  98% reduction
```

**The sandbox model:**
1. Heavy tool output is intercepted by hooks
2. Raw data goes into a local SQLite FTS5 index
3. Only a small summary enters the conversation
4. You can search the indexed data on demand with `ctx_search`

### Lab 4.3 — Session Continuity

Context Mode also tracks your work session. If the conversation compacts:

- File edits, git operations, and tasks are preserved in SQLite
- On compaction, a priority-tiered snapshot is built (≤2 KB)
- The model receives a **Session Guide** and picks up where it left off

**Try it:** Have a long conversation with several file edits, then ask Copilot to summarize what it's been working on. The session tracking keeps it informed.

---

## Part 5 — Microsoft Defender XDR Integration

### Overview

The **Defender XDR MCP extension** connects Copilot directly to your Microsoft 365 Defender tenant, enabling AI-assisted security operations. This is not just documentation lookup — these are **live response actions** against real incidents, devices, and identities.

### Prerequisites

- Microsoft 365 E5 or Microsoft Defender XDR license
- Appropriate RBAC permissions in the Defender portal
- The Defender MCP extension enabled in VS Code (available via GitHub Copilot)

### What Defender XDR Tools Provide

The Defender MCP integration exposes **22 tools** across four categories:

#### Category 1 — Incident Management (6 tools)

| Tool | Action | When to Use |
|------|--------|-------------|
| `defender_update_incident_status` | Set incident to active/resolved/redirected | Triaging or closing incidents |
| `defender_classify_incident` | Set classification (true positive, false positive, informational) and determination (malware, phishing, APT, etc.) | After investigation concludes |
| `defender_assign_incident` | Assign to an analyst by email | Routing during triage |
| `defender_add_incident_comment` | Add investigation notes | Documenting findings |
| `defender_add_incident_tags` | Tag for categorization | Organizing by campaign, team, priority |
| `defender_echo` | Test connectivity | Validating the MCP connection works |

#### Category 2 — Device Response Actions (8 tools)

| Tool | Action | When to Use |
|------|--------|-------------|
| `defender_get_machine_by_name` | Look up device by hostname | First step — get device details, risk score, health |
| `defender_isolate_device` | Network isolate (Full or Selective) | Contain active threat, prevent lateral movement |
| `defender_isolate_multiple` | Isolate multiple devices at once | Broad containment during active incident |
| `defender_release_device` | Remove network isolation | After remediation is complete |
| `defender_run_antivirus_scan` | Trigger Quick or Full AV scan | Suspicious activity on a device |
| `defender_stop_and_quarantine` | Kill process + quarantine file by SHA1 | Known malicious file running |
| `defender_restrict_code_execution` | Allow only Microsoft-signed code | Lock down compromised device |
| `defender_remove_code_restriction` | Remove code execution restrictions | After threat is resolved |

**Plus investigation tools:**

| Tool | Action | When to Use |
|------|--------|-------------|
| `defender_collect_investigation_package` | Collect forensic package (logs, system info) | Deep investigation needed |
| `defender_get_investigation_package_uri` | Get download URL for collected package | Downloading the forensic bundle |
| `defender_get_machine_actions` | List recent response actions (filter by device, type, status) | Checking action status/history |

#### Category 3 — Identity Response Actions (5 tools)

| Tool | Action | When to Use |
|------|--------|-------------|
| `defender_confirm_user_compromised` | Mark user as high risk in Entra ID Identity Protection | Confirmed credential compromise |
| `defender_confirm_user_safe` | Clear user risk (mark as safe) | After investigation confirms no compromise |
| `defender_disable_ad_account` | Disable AD account via Defender for Identity | Active credential theft, unauthorized access |
| `defender_enable_ad_account` | Re-enable a disabled AD account | After remediation |
| `defender_force_ad_password_reset` | Force password change at next logon | Credential theft detected (e.g., Mimikatz) |

#### Category 4 — Session Revocation (1 tool)

| Tool | Action | When to Use |
|------|--------|-------------|
| `defender_revoke_entra_sessions` | Revoke all Entra ID sign-in sessions and refresh tokens | Compromised credentials, force re-authentication |

---

### Lab 5.1 — Verify Defender XDR Connectivity

First, confirm the Defender MCP extension is connected:

**Prompt:**
> "Test the Defender MCP connection with an echo message"

This calls `defender_echo`. If it returns your message, you're connected to your tenant.

---

### Lab 5.2 — Device Investigation

**Scenario:** You receive an alert about suspicious activity on workstation `WS-FINANCE-01`.

**Step 1 — Look up the device:**
> "Look up the device WS-FINANCE-01 in Defender"

Copilot calls `defender_get_machine_by_name` and returns:
- Device ID, health status, OS version
- Risk score (Low/Medium/High/Critical)
- Exposure level
- Last seen timestamp

**Step 2 — Run an antivirus scan:**
> "Run a quick antivirus scan on WS-FINANCE-01 because of suspicious PowerShell activity"

Copilot calls `defender_run_antivirus_scan` with scan type `Quick`.

**Step 3 — Check the action status:**
> "Check the status of recent machine actions for WS-FINANCE-01"

Copilot calls `defender_get_machine_actions` filtered by device name.

---

### Lab 5.3 — Incident Triage with Copilot

**Scenario:** A new incident (ID: 42) appears — multi-stage attack with suspicious sign-ins and lateral movement.

**Step 1 — Assign the incident:**
> "Assign Defender incident 42 to analyst@contoso.com"

**Step 2 — Tag it for tracking:**
> "Add the tags 'active-investigation' and 'lateral-movement' to incident 42"

**Step 3 — Add investigation notes:**
> "Add a comment to incident 42: Initial triage started. Multiple devices showing beacon-like network patterns. Investigating scope of compromise."

**Step 4 — After investigation, classify:**
> "Classify incident 42 as a true positive with determination of multi-staged attack"

**Step 5 — Resolve when done:**
> "Resolve incident 42 with comment: Contained via device isolation and credential reset. Root cause was phishing email leading to Cobalt Strike deployment."

---

### Lab 5.4 — Identity Threat Response

**Scenario:** Alert indicates `jdoe@contoso.com`'s credentials were found in a credential dump.

**Step 1 — Mark the user as compromised:**
> "Mark jdoe@contoso.com as compromised in Defender — credentials found in dark web credential dump"

This triggers high-risk status in Entra ID Identity Protection, which activates Conditional Access policies (MFA challenge, block sign-in, etc.).

**Step 2 — Revoke all sessions:**
> "Revoke all Entra ID sessions for jdoe@contoso.com — forcing re-authentication across all devices"

**Step 3 — Force password reset:**
> "Force a password reset for jdoe@contoso.com due to credential compromise"

**Step 4 — After remediation, clear risk:**
> "Mark jdoe@contoso.com as safe in Defender — password has been reset and MFA re-enrolled"

---

## Part 6 — Incident Response Walkthrough

### Full Scenario: Phishing → Credential Theft → Lateral Movement

This walkthrough chains together multiple Defender tools in a realistic incident response. In a live demo, each prompt would be typed into Copilot Chat.

```
TIMELINE OF EVENTS:
  09:00  Phishing email received by jdoe@contoso.com
  09:15  User clicks malicious link → malware payload downloaded
  09:30  Mimikatz detected on WS-FINANCE-01
  09:35  Lateral movement to WS-EXEC-02 detected
  09:40  SOC analyst begins AI-assisted response
```

#### Phase 1 — Contain (9:40)

```text
Prompt: "Look up device WS-FINANCE-01 in Defender"
→ defender_get_machine_by_name: Returns device ID, risk = High

Prompt: "Isolate WS-FINANCE-01 and WS-EXEC-02 with full isolation — active lateral 
         movement from phishing compromise"
→ defender_isolate_multiple: Both devices isolated

Prompt: "Mark jdoe@contoso.com as compromised — Mimikatz credential theft detected"
→ defender_confirm_user_compromised: User risk set to High

Prompt: "Revoke all sessions for jdoe@contoso.com"
→ defender_revoke_entra_sessions: All tokens invalidated

Prompt: "Force password reset for jdoe@contoso.com — credentials stolen via Mimikatz"
→ defender_force_ad_password_reset: Password reset enforced
```

#### Phase 2 — Investigate (9:50)

```text
Prompt: "Run a full antivirus scan on WS-FINANCE-01"
→ defender_run_antivirus_scan (Full): Scan initiated

Prompt: "Collect an investigation package from WS-FINANCE-01 — need forensic data 
         for phishing compromise analysis"
→ defender_collect_investigation_package: Package collection started

Prompt: "Restrict code execution on WS-FINANCE-01 to Microsoft-signed only"
→ defender_restrict_code_execution: Only signed code allowed

Prompt: "Check the status of all recent actions on WS-FINANCE-01"
→ defender_get_machine_actions: Shows scan, package, restriction status
```

#### Phase 3 — Remediate & Document (10:15)

```text
Prompt: "Add comment to incident 42: Containment complete. Both workstations isolated. 
         User credentials reset. Forensic package collected from WS-FINANCE-01. 
         Awaiting AV scan results."
→ defender_add_incident_comment

Prompt: "Tag incident 42 with 'phishing', 'mimikatz', 'lateral-movement', 'contained'"
→ defender_add_incident_tags

Prompt: "Classify incident 42 as true positive — multi-staged attack"
→ defender_classify_incident
```

#### Phase 4 — Recover (after root cause confirmed)

```text
Prompt: "Release WS-EXEC-02 from isolation — confirmed clean after full scan"
→ defender_release_device

Prompt: "Remove code execution restrictions on WS-FINANCE-01 — reimaged and verified"
→ defender_remove_code_restriction

Prompt: "Mark jdoe@contoso.com as safe — password reset complete, MFA re-enrolled"
→ defender_confirm_user_safe

Prompt: "Enable the AD account for jdoe@contoso.com — remediation complete"
→ defender_enable_ad_account

Prompt: "Resolve incident 42: Root cause was phishing email with Cobalt Strike payload. 
         Contained via isolation and credential reset. Devices reimaged. 
         User re-onboarded with MFA."
→ defender_update_incident_status (resolved)
```

---

### Key Takeaways for Presentation

1. **MCP is modular** — add only the servers you need via `mcp.json`
2. **HTTP vs stdio** — remote APIs vs local processes, both work seamlessly
3. **Context optimization matters** — raw tool output can consume 40%+ of context; Context Mode reduces this by ~98%
4. **Defender XDR integration is operational** — these are real response actions, not just documentation lookups
5. **AI-assisted IR saves time** — natural language prompts replace clicking through multiple portal pages
6. **Audit trail built in** — every action requires a comment, creating a documented response timeline
7. **Chain tools for complex scenarios** — combine device, identity, and incident management in one conversation

---

## Appendix — Troubleshooting

### MCP Servers Not Showing in Copilot

- Ensure you're in **Agent mode** (not Ask or Edit mode)
- Check `.vscode/mcp.json` syntax — use a JSON validator
- Restart VS Code after editing `mcp.json`
- For stdio servers, verify the command is available: `npx @playwright/mcp@latest --help`

### Context Mode Not Working

- Verify installation: `context-mode --version`
- Check hooks file exists at `.github/hooks/context-mode.json`
- Run `ctx doctor` in a Copilot conversation for diagnostics

### Defender Tools Not Available

- Ensure the Defender MCP extension is installed and enabled
- Verify tenant connectivity with `defender_echo`
- Check RBAC permissions — you need Security Operator or Security Administrator role
- Some actions (device isolation, account disable) require elevated permissions

### Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| "Tool not found" | Server not registered or VS Code not restarted | Add to `mcp.json`, restart |
| "Connection refused" | HTTP server unreachable | Check URL, network, and firewall |
| "Permission denied" | Insufficient RBAC for Defender action | Elevate role in Entra ID |
| "Device not found" | Hostname doesn't match Defender inventory | Verify exact hostname in Defender portal |

---

## Final `mcp.json` Reference

After completing all labs, your configuration should look like:

```json
{
  "servers": {
    "context-mode": {
      "command": "context-mode",
      "type": "stdio"
    },
    "MicrosoftLearn": {
      "type": "http",
      "url": "https://learn.microsoft.com/api/mcp"
    },
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"],
      "type": "stdio"
    },
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "${input:context7ApiKey}"
      }
    }
  },
  "inputs": [
    {
      "id": "context7ApiKey",
      "type": "promptString",
      "description": "Context7 API Key (get one at https://context7.com)",
      "password": true
    }
  ]
}
```

> **Note:** The Defender XDR tools are provided by the Defender MCP extension built into VS Code/Copilot — they don't require a separate entry in `mcp.json`.
