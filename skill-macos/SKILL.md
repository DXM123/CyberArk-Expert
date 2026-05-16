---
name: cyberark-expert
description: "Unified CyberArk expert skill for troubleshooting, automation, and administration across all CyberArk products. Use when a user reports a CyberArk error message, asks about CyberArk configuration, needs API guidance, wants to automate tasks in Privilege Cloud or Identity portal, or asks anything CyberArk-related. Covers: Core PAS, Privilege Cloud, Identity (ISPSS), EPM, Remote Access, AAM, IGA, SCIM, Conjur, Secrets Manager, SCA, Secure Browser, DPA, PSM, CPM, PVWA. Triggers on: error codes (ITATS, CACPM, PVWA, CAVLT, etc.), CyberArk, Privilege Cloud, pCloud, Identity portal, PAM, EPM, PSM, CPM, PVWA, safe, platform, credential provider, conjur, psPAS, CyberArk API, known issue, workaround, CyberArk error."
---

# CyberArk Expert

## Overview

This skill provides a structured, source-driven workflow for:
1. **Error diagnosis** — error code lookup via official docs + community KI articles + web fallback
2. **API guidance** — routing to correct API documentation (Privilege Cloud, Identity, PAM Self-Hosted)
3. **Automation** — PowerShell (psPAS), Python SDK, REST API patterns, browser automation
4. **Portal administration** — Identity admin portal navigation, user/role/policy management
5. **Platform management** — Privilege Cloud platform operations (duplicate, export/import, activate)

**Golden rules:**
- ALWAYS check the local resolved-issues KB first: `~/.hermes/skills/cyberark-expert/resolved-issues.md`
- NEVER guess — every answer must have a source URL or explicit "I don't know"
- If no official source exists, provide a logic chain with Dual Confidence (e.g. ±15%, Medium-High)
- After resolving any issue, append it to the resolved-issues KB

---

## 1. Tenant Resolution

When a user asks you to perform actions against a CyberArk environment (API calls, browser navigation, account operations, etc.), you need to know **which environment** to target. The user registers their environments in:

```
~/.hermes/skills/cyberark-expert/tenants.md
```

### How it works

1. The user refers to a tenant by its **short ID** (e.g. "customer-one", "lab", "acme").
2. Read `tenants.md` and find the matching `## <short-id>` section.
3. Use the URLs from that section for all API calls, browser navigation, and doc lookups.
4. If the user mentions a tenant that doesn't exist in the file, ask them to provide the details — then offer to add it.

### Tenant fields

Each tenant entry contains:

| Field | Description | Example |
|---|---|---|
| **Name** | Display name for the environment | Acme Corp Production |
| **Type** | `Privilege Cloud` or `PAM Self-Hosted` | Privilege Cloud |
| **PVWA URL** | Web portal URL | `https://acme.privilegecloud.cyberark.cloud` |
| **Identity URL** | Identity admin portal (or `n/a` for self-hosted) | `https://abc1234.id.cyberark.cloud` |
| **PAM API Base** | REST API base URL | `https://acme.privilegecloud.cyberark.cloud/PasswordVault/API/` |
| **Notes** | Optional — CPM name, special config, contacts | CPM name: PasswordManager |

### Adding and updating tenants

The user can edit `tenants.md` manually, or ask you to do it:

- **"Add tenant acme with PVWA URL https://..."** → Append a new `## acme` section to `tenants.md` with the provided details. Ask the user for any missing fields.
- **"Update acme PVWA URL to https://..."** → Find the `## acme` section and update the relevant field.
- **"List my tenants"** → Read `tenants.md` and present a summary table.

When adding a new tenant, always use this format:

```markdown
## <short-id>
- **Name:** <display name>
- **Type:** <Privilege Cloud | PAM Self-Hosted>
- **PVWA URL:** <url>
- **Identity URL:** <url or n/a>
- **PAM API Base:** <url>
- **Notes:** <optional>

---
```

### When no tenant is specified

If the user asks for an action that requires a specific environment but doesn't name one:
- If only **one tenant** is registered in `tenants.md`, use it automatically.
- If **multiple tenants** exist, ask the user which one to use.
- If **no tenants** are registered, ask the user to provide the URL directly and offer to save it as a new tenant.

---

## 2. Error Code Diagnosis Workflow

When a user reports an error code or error message:

### Step 1: Check Local KB
Read `~/.hermes/skills/cyberark-expert/resolved-issues.md` and search for the error code or keywords. If a match is found, present the cached resolution and verify it's still current.

### Step 2: Identify the Component
Use the error code prefix to determine the CyberArk component:

| Prefix | Component |
|---|---|
| `APPAP` | Application Provider (Credential Provider) |
| `APPBC` | Application Provider (Backup/Alternate) |
| `CACPM` | Central Policy Manager |
| `CAS8N` | CASOS Internationalization |
| `CASGN` | CASOS General |
| `CASSM` | CASOS Session Management |
| `CASTM` | CASOS Transaction Management |
| `CAVLT` | CAVaultManager |
| `CVMVP` | CyberArk Digital Vault Cluster |
| `ENECONTROL` | Event Notification Engine — Controller |
| `ENEPR` | Event Notification Engine — Processing |
| `ITACM` | Vault Communication |
| `ITADB` | Vault Database Manager |
| `ITADM` | Vault Database Operations |
| `ITATE` | Vault Atomic Transactions |
| `ITATS` | Vault Application (Server-side) |
| `ITAWM` | Vault Workspace Monitor |
| `PADR` | Disaster Recovery |
| `PAREP` | Replicator |
| `PDKTC` | Password SDK (TCP) |
| `PSMIN` | PSM for SSH — Initialization |
| `PSMSH` | PSM for SSH — Shell / Commands Access Control |
| `PSMSR` | PSM Service |
| `PSMSV` | PSM Service (Log) |
| `PSPSH` | PSM for SSH — PSP |
| `PVWA` | Password Vault Web Access |
| `VCSS` | Vault-Conjur Synchronizer |
| `PASWS` | Privilege Cloud / PAM REST API |

Not all CyberArk errors follow the prefix convention. Privilege Cloud, Identity, SCA, EPM, and SWS errors may use different formats. If no prefix matches, proceed to Step 3 with the full error text.

### Step 3: Fetch Official Documentation

Based on the prefix, fetch the corresponding error messages page. Use **Chrome MCP** (not WebFetch — CyberArk docs are client-side rendered and return 404 via direct fetch).

#### PAM Self-Hosted — Digital Vault Server

| Component | Prefixes | URL |
|---|---|---|
| Communication | `ITACM` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/communication.htm |
| Application | `ITATS` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/application.htm |
| Database Operations | `ITADM`, `ITATE` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/database%20operations.htm |
| Database Manager | `ITADB` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/database%20manager.htm |
| Workspace Monitor | `ITAWM` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/workspace%20monitor.htm |
| CAVaultManager | `CAVLT` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/cavaultmanager.htm |
| CASOS | `CASGN`, `CASTM`, `CAS8N`, `CASSM` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/casos.htm |
| Event Notification Engine | `ENECONTROL`, `ENEPR` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/event%20notification%20engine.htm |
| Replicator | `PAREP` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/replicator.htm |
| Disaster Recovery | `PADR` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/disaster%20recovery%20messages.htm |
| Digital Vault Cluster | `CVMVP` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/cyberark%20digital%20cluster%20vault.htm |

#### PAM Self-Hosted — PAM Components

| Component | Prefixes | URL |
|---|---|---|
| PVWA | `PVWA` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/password%20vault%20web%20access%20messages.htm |
| CPM (index) | `CACPM` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/central%20policy%20manager%20messages.htm |
| CPM — General | `CACPM` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/central%20policy%20manager%20messages%20-%20general.htm |
| PSM Service | `PSMSR` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/psm%20service.htm |
| PSM Log | `PSMSV` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/log.htm |
| PSM for SSH / PSP | `PSPSH`, `PSMSH`, `PSMIN` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/psp.htm |
| Commands Access Control | `PSMSH`, `PSMIN` | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/commands%20access%20control.htm |

#### PrivateArk Client

| Component | URL |
|---|---|
| General | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/privateark%20client%20messages%20-%20general.htm |
| File and Workspace | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/file%20and%20workspace.htm |
| Object Messages | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/object%20messages.htm |
| Vault Properties / New Vault | https://docs.cyberark.com/pam-self-hosted/latest/en/content/messages/vault%20properties%20new%20vault.htm |

#### Credential Providers / Secrets Manager

| Component | Prefixes | URL |
|---|---|---|
| Application Provider | `APPAP`, `APPBC` | https://docs.cyberark.com/credential-providers/latest/en/content/messages/application%20provider%20messages%20-%20general.htm |
| Central Credential Provider | `APPAP`, `ITACM` | https://docs.cyberark.com/credential-providers/latest/en/content/cp%20for%20zos/activity-on-the-central-credential-provider.htm |
| Password SDK | `PDKTC` | https://docs.cyberark.com/credential-providers/latest/en/content/messages/password%20sdk%20messages%20-%20aam.htm |
| NETAIMGetAppInfo | — | https://docs.cyberark.com/credential-providers/latest/en/content/messages/netaimgetappinfo%20messages%20-%20aam.htm |
| Auditing | — | https://docs.cyberark.com/credential-providers/latest/en/content/messages/auditing%20messages%20-%20general.htm |
| Installation Troubleshoot | — | https://docs.cyberark.com/credential-providers/latest/en/content/messages/troubleshooting-installation.htm |

#### Other Products

| Product | URL |
|---|---|
| Secure Cloud Access (SCA) | https://docs.cyberark.com/sca/latest/en/content/troubleshooting/sca_error-codes.htm |
| Secure Infrastructure Access (SIA) | https://docs.cyberark.com/ispss-access/latest/en/content/admin/sia-windows-trblsht-eu-ref.htm |
| Secure Web Sessions (SWS) | https://docs.cyberark.com/sws/latest/en/content/setup/sws-errormessages.htm |
| EPM Sign-in Error Codes | https://docs.cyberark.com/epm/latest/en/content/epm/server%20user%20guide/viewsigninaudit.htm |
| Vault Synchronizer (Conjur) | https://docs.cyberark.com/conjur-enterprise/13.0/en/content/conjur/cv_conjur-vault-ts.htm |

#### Legacy Reference (single PDF, all PAM Self-Hosted errors v10.5)
https://docs.cyberark.com/pam-self-hosted/10.5/en/pdf/cyberark%20messages%20and%20responses%20guide.pdf

### Step 4: Search CyberArk Community

Search the community for the error code or keywords. The community search is URL-driven and works **without login**:

```
https://community.cyberark.com/s/global-search/<ERROR_CODE_OR_KEYWORDS>
```

The search results page has three tabs in the left sidebar:
- **Knowledge Articles** — structured KI articles with Symptom, Cause, Resolution
- **Documentation** — links to official docs
- **Discussions** — community threads (some may require login)

**KI article URL pattern:** `https://community.cyberark.com/s/article/<URL-Name>`

**KI article data shape:**
- Article Number (e.g. 000033504)
- Title
- Issue / Details (symptom description)
- Product
- Environment
- Cause
- Resolution (step-by-step fix)
- Related Versions
- Article Record Type

Use the available browser automation or web search integration to navigate, search results and read article content. Full KI articles are accessible without login.

### Step 5: Web Search Fallback

If official sources and community yield no results, search the web:
- Include the error code + "CyberArk" in the query
- Reddit (`r/CyberArk`), Stack Overflow, and vendor blogs often have useful threads
- Always note in the response that the source is unofficial

### Step 6: Log the Resolution

After resolving any issue, append to `~/.hermes/skills/cyberark-expert/resolved-issues.md`:

```markdown
## [YYYY-MM-DD] <ERROR_CODE> - <short description>
**Component:** <component name>
**Symptoms:** <what the user reported>
**Root cause:** <what was actually wrong>
**Resolution:** <step-by-step fix>
**Sources:** <URLs used>
---
```

### Output Format

Always structure error diagnosis responses as:

```
**PROBLEM:** [1-sentence summary]

**KNOWN ISSUES:**
- [Article #] [Title] — [article URL]
  Workaround: [from article or "see article"]
(or "No known issues found for these keywords.")

**RESOLUTION / WORKAROUND:**
[step-by-step, concrete actions]

**DOCUMENTATION:**
- [Product doc URL with section name]
```

---

## 3. API Documentation Routing

When a user asks about CyberArk APIs, route to the correct documentation based on the product:

### Privilege Cloud REST API (Gen 2/3)

**Primary docs:**
- https://docs.cyberark.com/privilege-cloud-shared-services/latest/en/content/webservices/implementing%20privileged%20account%20security%20web%20services%20.htm

**API URL structure:**
- Portal URL: `https://<subdomain>.cyberark.cloud/privilegecloud/`
- Gen 3 API: `https://<subdomain>.privilegecloud.cyberark.cloud/api/`
- Gen 2 API: `https://<subdomain>.privilegecloud.cyberark.cloud/PasswordVault/API/`
- Gen 1 API: `https://<subdomain>.privilegecloud.cyberark.cloud/PasswordVault/WebServices/`

**Auth:** Every API call (except Logon) requires `Authorization` header with session token.

**Throttling:** Privilege Cloud has API throttling — 429 errors when queue time is too long. Scripts should implement retry with short interval.

**Return codes:** 200 (Success), 201 (Created), 204 (No Content / DELETE success), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 409 (Conflict), 429 (Too Many Requests), 500 (Internal Server Error).

### Identity APIs

**Primary docs:**
- https://api-docs.cyberark.com/identity-docs-api/docs/identity-apis
- SCIM API: `https://<tenant-id>.id.cyberark.cloud/scim/`
- OAuth2: `POST https://<tenant-id>.id.cyberark.cloud/oauth2/token/<app-id>`

### PAM Self-Hosted API

**Primary docs:**
- https://docs.cyberark.com/pam-self-hosted/latest/en/content/webservices/implementing%20privileged%20account%20security%20web%20services%20.htm#ThePAMSelfHostedAPI

### API Quick Reference

**SCIM — list users:**
```bash
curl -H "Authorization: Bearer <token>" \
  "https://<tenant-id>.id.cyberark.cloud/scim/Users?filter=userName%20eq%20'user@domain'"
```

**PAM — list accounts:**
```bash
curl -H "Authorization: Bearer <token>" \
  "https://<tenant>.privilegecloud.cyberark.cloud/PasswordVault/API/Accounts"
```

**Get OAuth token:**
```bash
curl -X POST "https://<tenant-id>.id.cyberark.cloud/oauth2/token/<app-id>" \
  -d "grant_type=client_credentials&client_id=<id>&client_secret=<secret>"
```

---

## 4. Automation Tools

### psPAS — PowerShell Module for CyberArk PAM

For on-premise PAM or Privilege Cloud automation questions, reference:
- **Docs:** https://pspas.pspete.dev/commands/
- **Context7 ID:** `pspete/pspas` (1.7K snippets)

psPAS wraps the entire CyberArk PAM REST API in PowerShell cmdlets. Common patterns:
```powershell
# Connect to Privilege Cloud
New-PASSession -BaseURI "https://<subdomain>.privilegecloud.cyberark.cloud" -Type ISPSS

# Connect to on-premise PVWA
New-PASSession -BaseURI "https://pvwa.domain.com" -Credential $cred -Type CyberArk

# List safes
Get-PASSafe

# List accounts
Get-PASAccount -search "domain.com"

# Get password
Get-PASAccountPassword -AccountID <id>
```

### IdentityCommand — PowerShell Module for Identity

- **Context7 ID:** `pspete/identitycommand`
- Covers Identity API calls from PowerShell

### CyberArk Python SDK (ark-sdk-python)

- **Context7 ID:** `cyberark/ark-sdk-python`
- Official CyberArk SDK for Python-based automation

### CyberArk Ark SDK Golang

- **Context7 ID:** `cyberark/ark-sdk-golang`

### Bruno REST API Collections

- **Context7 ID:** `iam-jah/cyberark-rest-api-bruno` (68.2 score, 70 snippets)
- Ready-made API call collections for testing

---

## 5. Context7 MCP Integration

When the Context7 MCP is configured, use it to pull up-to-date documentation and code examples:

**Available CyberArk libraries on Context7:**

| Library | Context7 ID | Score | Snippets |
|---|---|---|---|
| CyberArk API | `api-docs.cyberark.com` | 54.8 | 80 |
| CyberArk Privilege Cloud | (docs.cyberark.com/privilege-cloud) | 41.2 | 684 |
| CyberArk PAM Self-Hosted | (docs.cyberark.com/pam-self-hosted) | 28.2 | 110 |
| CyberArk Identity Security Platform | (docs.cyberark.com) | 41.1 | 253 |
| CyberArk Identity APIs | (api-docs.cyberark.com/identity) | — | — |
| CyberArk Python SDK | `cyberark/ark-sdk-python` | — | 189 |
| CyberArk Identity Scripting | `lprzyb/cyberark-identity-dynamic-roles-attributes-js` | 79.1 | 172 |
| psPAS | `pspete/pspas` | — | 1,700 |
| IdentityCommand | `pspete/identitycommand` | — | 75 |
| Bruno Collections | `iam-jah/cyberark-rest-api-bruno` | 68.2 | 70 |
| Terraform Provider CyberArk | `cyberark/terraform-provider-cyberark` | — | 34 |
| Terraform Provider SIA | `aaearon/terraform-provider-cyberarksia` | 58.7 | 1,700 |
| Terraform Provider Idsec | `cyberark/terraform-provider-idsec` | 78.5 | 525 |
| Ark SDK Golang | `cyberark/ark-sdk-golang` | — | 168 |
| Grant CLI | `aaearon/grant-cli` | 91.3 | 216 |
| Aiobastion | `safepost/aiobastion` | 85.3 | 233 |
| CredentialRetriever | `pspete/credentialretriever` | — | 9 |

**Usage with Context7 MCP tools:**
1. `resolve-library-id` — resolve a library name to its Context7 ID
2. `query-docs` — fetch documentation and code examples by topic

---

## 6. CyberArk Documentation Tree (from llms.txt)

### Identity Security Platform Shared Services (ISPSS)
- [Overview & Deployment](https://docs.cyberark.com/ispss-deployment/latest/en/content/resources/_topnav/cc_home.htm)
- [Identity Administration](https://docs.cyberark.com/portal/latest/en/docs.htm)
- [Privileged Access to Resources](https://docs.cyberark.com/ispss-access/latest/en/content/resources/_topnav/cc_home.htm)
- [CORA AI](https://docs.cyberark.com/ai/latest/en/content/resources/_topnav/cc_home.htm)
- [CyberArk Blueprint](https://docs.cyberark.com/cyberark-blueprint/latest/en/content/resources/_topnav/cc_home.htm)

### Access and Identity Management
- [Identity](https://docs.cyberark.com/identity/latest/en/content/resources/_topnav/cc_home.htm)
- [Secure Web Sessions](https://docs.cyberark.com/sws/latest/en/content/resources/_topnav/cc_home.htm)
- [Identity Compliance](https://docs.cyberark.com/identity-compliance/latest/en/content/resources/_topnav/cc_home.htm)
- [Identity Flows](https://docs.cyberark.com/identity-flows/latest/en/content/resources/_topnav/cc_home.htm)
- [Workforce Password Management](https://docs.cyberark.com/wpm/latest/en/content/resources/_topnav/cc_home.htm)
- [CyberArk IGA](https://docs.cyberark.com/iga/latest/en/content/resources/_topnav/cc_home.htm)

### Privileged Access Management
- [PAM Self-Hosted](https://docs.cyberark.com/pam-self-hosted/latest/en/content/resources/_topnav/cc_home.htm)
- [Privilege Cloud](https://docs.cyberark.com/privilege-cloud-shared-services/latest/en/content/resources/_topnav/cc_home.htm)
- [Remote Access](https://docs.cyberark.com/remote-access-standard/latest/en/content/resources/_topnav/cc_home.htm)
- [Secure Infrastructure Access](https://docs.cyberark.com/dpa/latest/en/content/resources/_topnav/cc_home.htm)

### Endpoint Identity Security
- [Endpoint Privilege Manager](https://docs.cyberark.com/epm/latest/en/content/resources/_topnav/cc_home.htm)
- [Secure Browser](https://docs.cyberark.com/secure-browser/latest/en/content/resources/_topnav/cc_home.htm)

### Secrets Management
- [Conjur Enterprise](https://docs.cyberark.com/conjur-enterprise/latest/en/content/resources/_topnav/cc_home.htm)
- [Credential Providers](https://docs.cyberark.com/credential-providers/latest/en/content/resources/_topnav/cc_home.htm)
- [Conjur Cloud](https://docs.cyberark.com/conjur-cloud/latest/en/content/resources/_topnav/cc_home.htm)
- [Secrets Hub](https://docs.cyberark.com/secrets-hub-privilege-cloud/latest/en/content/resources/_topnav/cc_home.htm)

### Cloud Security
- [Secure Cloud Access](https://docs.cyberark.com/sca/latest/en/content/resources/_topnav/cc_home.htm)
- [Cloud Visibility](https://docs.cyberark.com/cloud-visibility/latest/en/content/resources/_topnav/cc_home.htm)

### Resources
- [Administration](https://docs.cyberark.com/admin-space/latest/en/content/resources/_topnav/cc_home.htm)
- [Audit](https://docs.cyberark.com/audit/latest/en/content/resources/_topnav/cc_home.htm)

**Important:** CyberArk documentation pages are client-side rendered. Use Chrome MCP to navigate and extract content. WebFetch returns 404 for these URLs.

---

## 7. Identity Portal Administration

Covers tenant URL patterns, login flows (browser UI + REST API bypass for the React SPA), portal architecture (iframe-based Angular shell with cross-origin content panels), navigation reference for all admin sections, common admin tasks (users, roles, policies, web apps, auth profiles), and key gotchas (policy inheritance, save behavior, iframe SecurityErrors, role naming conventions).

Read `references/identity-portal.md` for full details when performing Identity portal operations.

---

## 8. Privilege Cloud Operations

Covers API patterns for creating users (Identity context, username format requirements), creating safes (pcloud context, CPM assignment), adding users to safes (critical: role assignment must come first), and full platform management (list, duplicate, activate/deactivate, delete, and the export→modify INI→import cycle for password generation settings).

Read `references/privilege-cloud-ops.md` for full details when performing Privilege Cloud operations.

---

## 9. Browser Automation Patterns

Covers antixss token extraction (Identity iframe context), XSRF token extraction (Privilege Cloud context), and ExtJS 4.2.3 grid interaction patterns (component lookup, button triggering, virtualized grid access, active window detection).

Read `references/browser-automation.md` for code snippets when automating portal interactions.

---

## 10. MCP Servers for CyberArk

### CyberArk Secrets Manager MCP — Official
- **What:** Read and update Secrets Manager data, create workloads and secrets, scan for hardcoded secrets
- **Docs:** https://docs.cyberark.com/secrets-manager-saas/latest/en/content/conjurcloud/cc-mcp-server.htm
- **Requirements:** CyberArk Identity OAuth2 client and service account
- **Setup:** Create dedicated user → add to `Secrets Manager – Conjur Cloud User` role → create OAuth2 client

### Context7 MCP (@upstash/context7-mcp) — Documentation Fetcher
- **What:** Resolves library names to Context7 IDs, fetches up-to-date docs and code examples
- **Config:**
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
- **Claude Code:** `claude mcp add context7 -- npx -y @upstash/context7-mcp@latest`

---

## 11. Source Priority Chain

When answering any CyberArk question, follow this priority:

1. **Local resolved-issues KB** — `/Users/masko/.claude/skills/cyberark-expert/resolved-issues.md`
2. **Official CyberArk documentation** — docs.cyberark.com (use browser automation to fetch)
3. **CyberArk Community** — community.cyberark.com Knowledge Articles
4. **Context7 MCP** — for API docs and code examples
5. **psPAS / IdentityCommand docs** — for PowerShell automation
6. **Web search** — reddit, Stack Overflow, vendor blogs (mark as unofficial)

**NEVER present information from a lower-priority source without first checking higher-priority sources.**
