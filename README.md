# CyberArk Expert Skill for Claude

A unified CyberArk skill for Claude Desktop and Claude Code that provides structured, source-driven troubleshooting, automation guidance, and administration workflows across all CyberArk products.

## What It Does

- **Error diagnosis** — looks up error codes across official docs, CyberArk Community Knowledge Articles, and web sources, then caches resolutions locally for future reference
- **API routing** — points you to the correct documentation for Privilege Cloud, Identity, PAM Self-Hosted, SCIM, and OAuth2 APIs
- **Automation** — provides psPAS (PowerShell), Python SDK, REST API, and browser automation patterns
- **Portal administration** — guides you through Identity admin portal navigation, user/role management, and authentication configuration
- **Platform management** — covers Privilege Cloud platform operations including the export-modify-import cycle for password policies
- **Multi-tenant support** — register multiple CyberArk environments and reference them by name in conversation

## Supported Products

Core PAS, Privilege Cloud, Identity (ISPSS), EPM, Remote Access, AAM, IGA, SCIM, Conjur, Secrets Manager, SCA, Secure Browser, DPA, PSM, CPM, PVWA.

## Choose Your Platform

The skill is available in two platform-specific packages. Pick the one that matches the OS where you run Claude Desktop or Claude Code.

| Platform | Folder | Status |
|---|---|---|
| **macOS** | [`skill-macos/`](skill-macos/) | Tested and stable |
| **Windows** | [`skill-windows/`](skill-windows/) | Not yet tested — see note below |

Each folder contains the complete skill (SKILL.md, references, tenants, resolved-issues KB) along with a platform-specific README with detailed installation instructions.

### Windows — Not Yet Tested

The Windows version was built based on official Claude documentation and confirmed path conventions (`~/.claude/skills/` resolves to `%USERPROFILE%\.claude\skills\` on Windows). However, it has **not been tested on an actual Windows machine** yet. If you're the first Windows user, you may encounter issues that need adjustment — particularly around:

- MSIX installations redirecting config file paths (documented in the Windows README with a detection script)
- Filesystem MCP path format (forward slashes vs backslashes in allowed directories)
- Chrome MCP behavior on Windows for accessing CyberArk docs

If you run into problems, please open an issue. Fixes and corrections from Windows users are welcome.

## Quick Start

### 1. Copy the skill to your Claude skills directory

**macOS / Linux:**
```bash
cp -r skill-macos/ ~/.claude/skills/cyberark-expert/
```

**Windows (PowerShell):**
```powershell
Copy-Item -Path "skill-windows\*" -Destination "$env:USERPROFILE\.claude\skills\cyberark-expert\" -Recurse -Force
```

### 2. Restart Claude Desktop

Close and reopen Claude Desktop. The skill should appear in your skills list.

### 3. Start using it

Just mention anything CyberArk-related in conversation:

- *"I'm getting error ITATS531E..."*
- *"How do I create a safe in Privilege Cloud?"*
- *"Show me the psPAS command for listing accounts"*
- *"Help me navigate the Identity admin portal"*

### 4. Read the full setup guide

The platform-specific READMEs cover MCP server configuration (Context7, Filesystem permissions), multi-tenant setup, and optional communication rules that improve Claude's output quality:

- **macOS:** [`skill-macos/README.md`](skill-macos/README.md)
- **Windows:** [`skill-windows/README.md`](skill-windows/README.md)

## Repository Structure

```
cyberark-expert/
├── README.md                              # This file
├── skill-macos/                           # macOS skill package
│   ├── SKILL.md                           # Skill definition
│   ├── README.md                          # macOS setup guide
│   ├── tenants.md                         # Tenant registry
│   ├── resolved-issues.md                 # Local KB
│   └── references/
│       ├── identity-portal.md
│       ├── privilege-cloud-ops.md
│       └── browser-automation.md
└── skill-windows/                         # Windows skill package
    ├── SKILL.md                           # Skill definition (portable paths)
    ├── README.md                          # Windows setup guide
    ├── PLAN.md                            # Development planning notes
    ├── tenants.md                         # Tenant registry
    ├── resolved-issues.md                 # Local KB
    └── references/
        ├── identity-portal.md
        ├── privilege-cloud-ops.md
        └── browser-automation.md
```

## Prerequisites

- **Claude Desktop** or **Claude Code** installed
- **Node.js** — required for the Context7 MCP server (recommended). Download from https://nodejs.org
- **Chrome browser** with the Claude in Chrome extension — needed to access CyberArk documentation pages (they are client-side rendered and return 404 when fetched directly)

## How the Error Lookup Works

```
User reports error
    │
    ▼
Check local KB (resolved-issues.md)
    │ found? → present cached resolution
    │ not found? ▼
Identify component from error prefix (27 known prefixes)
    │
    ▼
Fetch official docs (Chrome MCP — client-side rendered pages)
    │
    ▼
Search CyberArk Community Knowledge Articles
    │
    ▼
Web search fallback (Reddit, Stack Overflow, blogs)
    │
    ▼
Present resolution with sources
    │
    ▼
Cache in resolved-issues.md for next time
```

## Contributing

If you find issues — especially on Windows — or want to add support for additional CyberArk products, error codes, or automation patterns, contributions are welcome.
