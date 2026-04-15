# CyberArk Expert Skill — Setup Guide (Windows)

## What This Is

A unified CyberArk skill for Claude Desktop / Claude Code on Windows.
It provides:

- Error code diagnosis with multi-source lookup chain
- API documentation routing across all CyberArk products
- Automation guidance (psPAS, Python SDK, REST API, browser automation)
- Identity portal administration playbook
- Privilege Cloud operations (platforms, safes, users)
- Local resolved-issues knowledge base

## File Structure

```
%USERPROFILE%\.claude\skills\cyberark-expert\    (after installation)
├── SKILL.md                  # The skill definition (main file)
├── README.md                 # This file
├── tenants.md                # Your CyberArk environments (PVWA URLs, tenant IDs, etc.)
├── resolved-issues.md        # Local KB of resolved issues (append-only)
└── references\               # Detailed operational guides (loaded on demand)
    ├── identity-portal.md    # Identity admin portal: login, navigation, tasks, gotchas
    ├── privilege-cloud-ops.md # Privilege Cloud: users, safes, platforms, API patterns
    └── browser-automation.md # Token extraction, ExtJS interaction patterns
```

## Setting Up Your Tenants

The skill supports **multiple CyberArk environments**. Instead of hardcoding a single PVWA URL, you register your environments in `tenants.md` and reference them by name in conversation.

### How it works

1. Open `tenants.md` and add your CyberArk environments (or ask the agent to do it for you).
2. In conversation, refer to the environment by its short ID: *"list accounts for acme"*, *"check error on lab-environment"*, *"create a safe in customer-two"*.
3. The agent looks up the matching entry in `tenants.md` and uses the registered URLs.

### Adding tenants manually

Open `tenants.md` and add a block for each environment. Delete the example entry and use this format:

```markdown
## acme-prod
- **Name:** Acme Corp Production
- **Type:** Privilege Cloud
- **PVWA URL:** https://acme.privilegecloud.cyberark.cloud
- **Identity URL:** https://abc1234.id.cyberark.cloud
- **PAM API Base:** https://acme.privilegecloud.cyberark.cloud/PasswordVault/API/
- **Notes:** CPM name: PasswordManager. Contact: admin@acme.com

---
```

For PAM Self-Hosted environments where there is no Identity portal:

```markdown
## internal-dc
- **Name:** Internal Data Center
- **Type:** PAM Self-Hosted
- **PVWA URL:** https://pvwa.internal.domain.com
- **Identity URL:** n/a
- **PAM API Base:** https://pvwa.internal.domain.com/PasswordVault/API/
- **Notes:** On-prem vault, VPN required

---
```

### Adding tenants via the agent

You can also ask the agent to add or update tenants in conversation:

- *"Add tenant acme-prod with PVWA URL https://acme.privilegecloud.cyberark.cloud and Identity URL https://abc1234.id.cyberark.cloud, type Privilege Cloud"*
- *"Update acme-prod PVWA URL to https://acme-new.privilegecloud.cyberark.cloud"*
- *"List my tenants"*
- *"Remove the example-lab tenant"*

The agent will read and write `tenants.md` directly.

### Field reference

| Field | Required | Description |
|---|---|---|
| **Short ID** | Yes | The `## <id>` heading — what you use in conversation to reference this tenant |
| **Name** | Yes | Human-readable display name |
| **Type** | Yes | `Privilege Cloud` or `PAM Self-Hosted` |
| **PVWA URL** | Yes | Web portal URL for this environment |
| **Identity URL** | No | Identity admin portal URL (set to `n/a` for self-hosted) |
| **PAM API Base** | Yes | REST API base URL |
| **Notes** | No | Anything else — CPM name, VPN requirements, contact person, etc. |

### Behavior when no tenant is specified

- **One tenant registered:** The agent uses it automatically.
- **Multiple tenants:** The agent asks which one you mean.
- **No tenants:** The agent asks you to provide a URL and offers to save it.

## Installation

### Prerequisites

- **Claude Desktop for Windows** — installed and running
- **Node.js** — required for Context7 MCP. Download from https://nodejs.org. After installation, verify `npx` is available by running `npx --version` in PowerShell.

### 1. Install the Skill

Copy the `skill-windows\` folder to your Claude skills directory.

**PowerShell:**
```powershell
# Option A: Copy
Copy-Item -Path "C:\path\to\cyberark-expert\skill-windows\*" -Destination "$env:USERPROFILE\.claude\skills\cyberark-expert\" -Recurse -Force

# Option B: Create the directory first if it doesn't exist
New-Item -ItemType Directory -Path "$env:USERPROFILE\.claude\skills\cyberark-expert" -Force
Copy-Item -Path "C:\path\to\cyberark-expert\skill-windows\*" -Destination "$env:USERPROFILE\.claude\skills\cyberark-expert\" -Recurse -Force
```

**Command Prompt:**
```cmd
xcopy /E /I "C:\path\to\cyberark-expert\skill-windows" "%USERPROFILE%\.claude\skills\cyberark-expert"
```

### 2. Remove Old Skills (if applicable)

Delete or disable the old `cyberark-supportagent` and `cyberark-identity` skills:

**PowerShell:**
```powershell
Remove-Item -Path "$env:USERPROFILE\.claude\skills\cyberark-supportagent" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path "$env:USERPROFILE\.claude\skills\cyberark-identity" -Recurse -Force -ErrorAction SilentlyContinue
```

### 3. Configure MCP Servers (Optional but Recommended)

Add these to your Claude Desktop `claude_desktop_config.json` or Claude Code MCP config.

#### Finding your config file on Windows

The config file location depends on how Claude Desktop was installed:

**Standard installation (.exe installer):**
```
%APPDATA%\Claude\claude_desktop_config.json
```

**MSIX installation (Microsoft Store, WinGet, enterprise MSIX):**
```
%LOCALAPPDATA%\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\claude_desktop_config.json
```

**Important:** MSIX installations silently redirect `%APPDATA%\Claude\` to the virtualized path above. The "Edit Config" button in Claude Desktop's Developer settings may open the wrong file. If your MCP servers aren't loading after editing the config, check the MSIX path instead.

**To find which path your installation uses, run this in PowerShell:**
```powershell
# Check if MSIX virtualized path exists
$msixPath = "$env:LOCALAPPDATA\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\claude_desktop_config.json"
$standardPath = "$env:APPDATA\Claude\claude_desktop_config.json"

if (Test-Path $msixPath) {
    Write-Host "MSIX install detected. Config file at:" -ForegroundColor Yellow
    Write-Host $msixPath
} elseif (Test-Path $standardPath) {
    Write-Host "Standard install detected. Config file at:" -ForegroundColor Green
    Write-Host $standardPath
} else {
    Write-Host "Config file not found at either location." -ForegroundColor Red
}
```

#### Context7 — Documentation Fetcher (Highly Recommended)

Provides on-demand access to CyberArk documentation and code examples.

**Claude Desktop (`claude_desktop_config.json`):**
```json
{
  "mcpServers": {
    "Context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

**Claude Code:**
```powershell
claude mcp add context7 -- npx -y @upstash/context7-mcp@latest
```

No authentication needed for basic usage. For higher rate limits, get an API key from https://context7.com/dashboard.

#### CyberArk Secrets Manager MCP — Official (Optional)

For Conjur Cloud secrets management. Requires Identity OAuth2 client setup.

**Docs:** https://docs.cyberark.com/secrets-manager-saas/latest/en/content/conjurcloud/cc-mcp-server.htm

**Setup steps:**
1. Create a dedicated user in Identity Administration
2. Add user to `Secrets Manager – Conjur Cloud User` role
3. Create an OAuth2 confidential client in Identity
4. Configure the MCP server with the client ID, secret, and tenant URL

#### Filesystem MCP — Allowed Directories (Required for Local KB)

The skill writes to `resolved-issues.md` and `tenants.md` inside its own installation directory. If you have the Filesystem MCP server configured in Claude Desktop, its `args` array must include the skill's parent path — otherwise writes will be rejected with **"path outside allowed directories"**.

Add your skills directory to your Filesystem MCP configuration:

**Example (`claude_desktop_config.json`):**
```json
"filesystem": {
  "command": "npx",
  "args": [
    "-y",
    "@anthropic-ai/mcp-filesystem@latest",
    "C:/Users/<your-username>/.claude/skills"
  ]
}
```

**Note:** Use forward slashes (`/`) in the JSON config even on Windows — Node.js and the MCP server handle them correctly. If you already have other allowed directories listed in `args`, just append the skills path as an additional entry — don't remove existing ones.

**Why this is needed:** When you report a CyberArk error and the agent resolves it, the resolution gets appended to `resolved-issues.md` for future reference. Similarly, when you add or update tenants via conversation, the agent writes to `tenants.md`. Both operations require filesystem write access to the skill's installation directory.

### 4. Recommended Communication Rules (Optional)

This skill works on its own — you don't need any additional configuration. However, the skill was designed alongside a set of **global communication rules** that shape how Claude responds in general. Applying these rules can significantly improve the quality and consistency of Claude's output, especially in technical troubleshooting and CyberArk work.

**These rules are not required**, but if you want Claude to behave in a more direct, fact-driven, no-bullshit way, you can add them to your environment. They are separated from the skill on purpose — they affect all conversations, not just CyberArk topics.

#### The Rules

Copy the block below and paste it into the appropriate location for your Claude environment (see placement guide after the block):

```markdown
# Communication Rules

## Tone
- Keep it casual and direct in chat. Swearing is fine when things are broken or frustrating.
- Professional tone ONLY when preparing deliverables: emails, documents, presentations, reports.
- Omit apologies, disclaimers, and formalities. Get to the point.

## Accuracy
- Zero tolerance for assumptions and made-up information. Every claim must be backed by a source.
- If you make an assumption, explicitly label it as such — never let it blend in with facts.
- If something is unknown, say "I don't know" — do not fabricate an answer.
- If no source exists, provide a logic chain and Dual Confidence rating (e.g. ±15%, Medium-High).

## Quality Control
- "Keyword check" triggers a recursive review of logic, metric compliance, and grammar.
- Always question your own decisions — think twice about relevance before presenting results.
- Provide direct URLs only. No blind recommendations without a link to the source.

## Structure
- Prioritize core intent. Answer what was actually asked before expanding.
- Break complex topics into sequential steps.
- Use Markdown (##, ###, tables) for scannability.

## Tools and Files Access
- When you need to access some files, tools and/or other interaction from the user, do not look for any workarounds. Stop your work and ask the user to provide assistance or instructions.
```

#### Where to Place These Rules

The right location depends on which Claude environment you use. You can apply them in multiple places — they don't conflict.

**Claude Code (terminal on Windows):**

For rules that apply to ALL your Claude Code projects globally:
```
%USERPROFILE%\.claude\CLAUDE.md
```
Append the rules block to this file (create it if it doesn't exist). Every Claude Code session on your machine will pick them up automatically.

For rules that apply only to a specific project:
```
<your-project-root>\.claude\CLAUDE.md
```
Place the rules in the project's `.claude\CLAUDE.md` file. They apply only when Claude Code is run from that project directory. Project-level rules are merged with global rules — project takes precedence if there's a conflict.

**Cowork (Claude Desktop app — Cowork mode):**

Cowork reads from the `.claude\CLAUDE.md` file inside the folder you select when starting a session. So if you select `C:\Users\you\my-project\` as your Cowork folder, the rules should be in:
```
C:\Users\you\my-project\.claude\CLAUDE.md
```

If you want the rules to apply across multiple Cowork sessions with different folders, you'll need to place a `.claude\CLAUDE.md` in each folder — or keep one master copy and create symbolic links:

```powershell
# Create master rules file
New-Item -ItemType Directory -Path "$env:USERPROFILE\.claude" -Force
# Edit %USERPROFILE%\.claude\CLAUDE.md and add your rules

# Create symbolic links into each project (requires elevated PowerShell)
New-Item -ItemType SymbolicLink -Path "C:\path\to\project-a\.claude\CLAUDE.md" -Target "$env:USERPROFILE\.claude\CLAUDE.md"
New-Item -ItemType SymbolicLink -Path "C:\path\to\project-b\.claude\CLAUDE.md" -Target "$env:USERPROFILE\.claude\CLAUDE.md"
```

**Note:** Creating symbolic links on Windows requires running PowerShell as Administrator, or having Developer Mode enabled in Windows Settings.

**Claude.ai (web chat):**

The web chat at claude.ai does not read `CLAUDE.md` files. To apply these rules in web chat, you have two options:
1. **Project instructions** — Create a Project in claude.ai and paste the rules into the project's custom instructions field. They will apply to all conversations within that project.
2. **Manual prompt** — Paste the rules block at the start of a conversation, or save them as a prompt template you can reuse.

#### Summary Table

| Environment | File Location | Scope |
|---|---|---|
| Claude Code (global) | `%USERPROFILE%\.claude\CLAUDE.md` | All projects on this machine |
| Claude Code (project) | `<project>\.claude\CLAUDE.md` | Single project only |
| Cowork | `<selected-folder>\.claude\CLAUDE.md` | Single Cowork session folder |
| Claude.ai (web) | Project custom instructions | Single project in claude.ai |
| Claude.ai (web) | Paste at conversation start | Single conversation |

## Usage

Once installed, the skill triggers automatically when you mention CyberArk-related topics:

- "I'm getting error ITATS531E..."
- "How do I create a safe in Privilege Cloud?"
- "Show me the psPAS command for listing accounts"
- "What's the API endpoint for platform management?"
- "Help me navigate the Identity admin portal"

## How the Error Lookup Works

```
User reports error
    │
    ▼
Check local KB (resolved-issues.md)
    │ found? → present cached resolution
    │ not found? ▼
Identify component from error prefix
    │
    ▼
Fetch official docs page (Chrome MCP — docs are client-side rendered)
    │
    ▼
Search CyberArk Community (URL: community.cyberark.com/s/global-search/<query>)
    │
    ▼
If still no answer → web search (Reddit, Stack Overflow, blogs)
    │
    ▼
Present resolution with sources
    │
    ▼
Append to resolved-issues.md for future reference
```

## Notes

- CyberArk documentation pages (docs.cyberark.com) are **client-side rendered** — they return 404 when fetched directly. Use Chrome MCP or browser tools to access them.
- CyberArk Community search works **without login** for Knowledge Articles. Some Discussions may require authentication.
- The resolved-issues KB is a single append-only markdown file. Search it by error code, component, or keywords.
- **Windows MSIX installations** may redirect config file paths silently. If MCP servers don't load, check the MSIX virtualized path (see Section 3 above).
- **psPAS** is a native PowerShell module — Windows users can use it directly without any additional shell setup.
