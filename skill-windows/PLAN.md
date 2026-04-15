# CyberArk Expert Skill — Windows Adaptation Plan

## 1. What Can Be Reused As-Is (No Changes Needed)

These sections of the macOS skill are platform-independent and can be copied directly:

| Section | Reason |
|---|---|
| **Frontmatter** (name, description) | Identical — skill triggers are platform-agnostic |
| **Section 1: Tenant Resolution** | Logic is the same; only file paths change |
| **Section 2: Error Code Diagnosis Workflow** | Error prefixes, doc URLs, community search, output format — all identical |
| **Section 3: API Documentation Routing** | URLs and API patterns are platform-independent |
| **Section 4: Automation Tools** | psPAS, Python SDK, Bruno — all cross-platform. PowerShell examples already use psPAS which is native to Windows |
| **Section 5: Context7 MCP Integration** | Library IDs and MCP tools are the same |
| **Section 6: CyberArk Documentation Tree** | Static URL list, no platform dependency |
| **Section 7: Identity Portal Administration** | Browser-based operations, platform-independent |
| **Section 8: Privilege Cloud Operations** | API-based operations, platform-independent |
| **Section 10: MCP Servers for CyberArk** | MCP config format is the same (JSON) |
| **Section 11: Source Priority Chain** | Logic chain, only file paths change |
| **references/identity-portal.md** | Entirely browser/API content |
| **references/privilege-cloud-ops.md** | Entirely API content |

**Bottom line:** ~90% of the SKILL.md content is reusable. The changes are concentrated in file paths, browser automation patterns, and the README installation instructions.

---

## 2. What Must Change

### 2.1. File Paths in SKILL.md

Every hardcoded path in the macOS skill uses Unix-style paths pointing to the macOS global skill location. These must be updated to Windows equivalents.

| macOS Path | Windows Path |
|---|---|
| `/Users/masko/.claude/skills/cyberark-expert/` | `~\.claude\skills\cyberark-expert\` |
| `/Users/masko/.claude/skills/cyberark-expert/resolved-issues.md` | `~\.claude\skills\cyberark-expert\resolved-issues.md` |
| `/Users/masko/.claude/skills/cyberark-expert/tenants.md` | `~\.claude\skills\cyberark-expert\tenants.md` |

**CONFIRMED (from official docs):** Claude Code and Claude Desktop use `~/.claude/` as the user-scope directory on ALL platforms. On Windows, `~` resolves to `%USERPROFILE%` (typically `C:\Users\<username>`). The official documentation consistently uses `~/.claude/skills/` with forward slashes — this works cross-platform because Claude's tooling normalizes paths internally.

**Decision:** Use `~/.claude/skills/cyberark-expert/` with forward slashes in SKILL.md. This is the same notation the official docs use and avoids the need for user-specific path replacement. The macOS version's hardcoded `/Users/masko/...` path was user-specific anyway — the Windows version should use the portable `~/` notation, and ideally the macOS version should be updated to match.

**Affected locations in SKILL.md:**
- Golden rules (line 18–21)
- Section 1: tenants.md path (line 30)
- Section 2, Step 1: resolved-issues.md path (line 89)
- Section 2, Step 6: resolved-issues.md path (line 229)
- Section 11: resolved-issues.md path (line 494)

### 2.2. Browser Automation Patterns (Section 9 + references/browser-automation.md)

The JavaScript snippets for token extraction and ExtJS interaction are **browser-level** and technically platform-independent (they run inside Chrome DevTools / Chrome MCP regardless of OS). However:

- **Chrome MCP availability on Windows** — needs verification. The `mcp__Claude_in_Chrome__*` tools and `mcp__Control_Chrome__*` tools should work if the Chrome extension is installed, but this must be confirmed.
- **Computer-use MCP on Windows** — Claude Desktop's computer-use tools (`mcp__computer-use__*`) may behave differently on Windows (different app tiers, different screenshot format, different keyboard shortcuts). This section may need Windows-specific notes.
- **Browser paths** — if any automation references browser binary paths or profile paths, these differ on Windows:
  - macOS: `/Applications/Google Chrome.app/`
  - Windows: `C:\Program Files\Google\Chrome\Application\chrome.exe`

**Action:** Review `references/browser-automation.md` — current content is pure JavaScript (cookie extraction, ExtJS component queries). These are browser-engine-level and should work unchanged. Add a note about Windows-specific Chrome MCP setup if needed.

### 2.3. README.md — Full Rewrite Required

The README is almost entirely platform-specific. A new Windows README must cover:

#### Installation
- **Copy command:** `xcopy` or `robocopy` instead of `cp -r`, or PowerShell `Copy-Item`
- **Symlink command:** `mklink /D` instead of `ln -s` (requires elevated prompt)
- **Skill path:** `%USERPROFILE%\.claude\skills\cyberark-expert\`
- **Old skill removal:** `rmdir /s /q` or `Remove-Item -Recurse` instead of `rm -rf`

#### Claude Desktop Config Location
- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

#### MCP Server Configuration
- Context7 config: same JSON, but the README should show the Windows config file path
- Filesystem MCP allowed directories: Windows paths (`C:\Users\<username>\.claude\skills`)
- npx command: same, but user needs Node.js installed and in PATH on Windows

#### Communication Rules Placement
- **Claude Code on Windows:** `%USERPROFILE%\.claude\CLAUDE.md`
- **Cowork on Windows:** `<selected-folder>\.claude\CLAUDE.md`
- **Symlink instructions:** `mklink` instead of `ln -s`

### 2.4. tenants.md and resolved-issues.md

These are pure Markdown data files — content is identical. Can be copied as-is.

---

## 3. New Windows-Specific Considerations

### 3.1. PowerShell as Primary Shell

On Windows, Claude Desktop's Bash tool runs in a different environment. The skill should:
- Prefer PowerShell syntax for any shell examples in the README
- Note that psPAS is **natively PowerShell** — Windows users have a natural advantage here
- Include PowerShell equivalents for any bash commands in the skill body

### 3.2. Claude Desktop on Windows — Feature Parity (Confirmed)

Based on official documentation research:

| Feature | Status | Notes |
|---|---|---|
| Skill loading from `~/.claude/skills/` | **CONFIRMED** | Same `~/.claude/` directory structure on all platforms. On Windows `~` = `%USERPROFILE%` |
| Skills via ZIP upload | **CONFIRMED** | Customize > Skills > Upload — works identically on Windows |
| Chrome MCP (Claude in Chrome) | **CONFIRMED** | Chrome extension is cross-platform; same tools on Windows |
| Control Chrome MCP | **CONFIRMED** | Cross-platform Chrome DevTools protocol |
| Computer-use MCP | **CONFIRMED** | Available on Windows Claude Desktop (Cowork requires Virtual Machine Platform feature enabled) |
| Filesystem MCP | **CONFIRMED** | Cross-platform; uses forward slashes in config even on Windows |
| Context7 MCP (npx) | **CONFIRMED** | Node.js + npx is cross-platform; same config JSON |
| CLAUDE.md loading | **CONFIRMED** | `~/.claude/CLAUDE.md` for global, `.claude/CLAUDE.md` for project — same on all platforms |

### 3.3. Cowork on Windows

- **CONFIRMED:** Cowork is available on Windows Claude Desktop. Requires the **Virtual Machine Platform** Windows feature to be enabled.
- File tools (Read/Write/Edit) work with Windows paths — Claude's sandbox handles path normalization.

### 3.4. Claude Desktop Config File — MSIX Gotcha

There is a **known issue** with Windows MSIX installations (Microsoft Store, WinGet, enterprise MSIX):

- **Documented path:** `%APPDATA%\Claude\claude_desktop_config.json`
- **Actual MSIX path:** `%LOCALAPPDATA%\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\claude_desktop_config.json`

The MSIX container silently redirects `%APPDATA%\Claude\` to the virtualized path. The "Edit Config" button in Developer settings may open the wrong file. The README must warn about this and explain how to find the correct config file on MSIX installations.

---

## 4. Implementation Steps

### Phase 1: Research & Validation — COMPLETED
1. **Skills directory:** `~/.claude/skills/` on all platforms. On Windows, `~` = `%USERPROFILE%`. ✅
2. **Config path:** `%APPDATA%\Claude\claude_desktop_config.json` (standard) or MSIX virtualized path. ✅
3. **Chrome MCP:** Cross-platform, works on Windows with Chrome extension installed. ✅
4. **Filesystem MCP:** Cross-platform, forward slashes work in config. ✅
5. **Path format:** Official docs use `~/.claude/` with forward slashes everywhere — this is the portable notation. ✅

### Phase 2: Create SKILL.md (Windows)
1. Copy macOS SKILL.md as base
2. Replace all file paths with Windows equivalents
3. Add Windows-specific notes to Section 9 (browser automation) if needed
4. Verify line count stays under 500

### Phase 3: Create references/ (Windows)
1. Copy `identity-portal.md` — likely unchanged
2. Copy `privilege-cloud-ops.md` — likely unchanged
3. Copy or adapt `browser-automation.md` — add Windows notes if Chrome MCP behaves differently

### Phase 4: Create README.md (Windows)
1. Write from scratch (not a find-replace of the macOS README)
2. All installation commands in PowerShell
3. Windows config file paths
4. Windows-specific MCP setup (filesystem allowed dirs with backslashes)
5. Communication rules placement for Windows Claude Code / Cowork / claude.ai
6. Filesystem MCP permissions section (Windows paths)

### Phase 5: Create Supporting Files
1. `tenants.md` — copy from macOS (identical)
2. `resolved-issues.md` — copy from macOS (identical)

### Phase 6: Testing
1. Install the skill on a Windows machine
2. Verify it appears in Claude Desktop's skill list
3. Test error lookup workflow end-to-end
4. Test tenant management (add, list, update)
5. Test resolved-issues KB write
6. Test Context7 MCP integration
7. Test browser-based CyberArk doc lookup

---

## 5. File Structure (Target)

```
skill-windows/
├── SKILL.md                  # Windows-adapted skill definition
├── README.md                 # Windows-specific setup guide
├── tenants.md                # Identical to macOS version
├── resolved-issues.md        # Identical to macOS version
└── references/
    ├── identity-portal.md    # Identical to macOS version
    ├── privilege-cloud-ops.md # Identical to macOS version
    └── browser-automation.md # Possibly adapted for Windows
```

---

## 6. Risk Assessment (Updated After Research)

| Risk | Impact | Mitigation |
|---|---|---|
| MSIX config path mismatch | User edits wrong config file, MCP servers don't load | Document both paths in README; explain how to detect MSIX install |
| `~/` tilde not expanding in SKILL.md on some Windows setups | Paths in golden rules / KB references break | Provide fallback instructions to use literal `C:\Users\<username>\...` paths |
| User installs Node.js but npx not in PATH | Context7 MCP fails to start | Include PATH verification step in README |
| Filesystem MCP backslash vs forward slash confusion | Allowed directories not matching | Document that forward slashes work in JSON config on Windows |

---

## 7. Key Insight: Consider a Single Cross-Platform Skill

Research revealed that the official Claude documentation uses `~/.claude/skills/` with forward slashes on ALL platforms. This means the macOS SKILL.md could be made cross-platform by replacing the hardcoded `/Users/masko/...` paths with `~/.claude/skills/cyberark-expert/...`.

**Two options going forward:**

**Option A: Two separate skills (current plan)**
- `skill-macos/` — keeps hardcoded macOS paths (current state)
- `skill-windows/` — uses `~/` portable paths
- Separate READMEs for each platform
- Pro: Each README is focused and clear
- Con: Maintaining two SKILL.md files that are 95% identical

**Option B: One cross-platform skill + platform-specific READMEs**
- Single `skill/SKILL.md` using `~/.claude/skills/cyberark-expert/` paths (works on both)
- `README-macos.md` and `README-windows.md` (or one README with platform sections)
- Pro: Single source of truth for the skill logic
- Con: README is slightly more complex

**Recommendation:** Option B is cleaner long-term. The only real platform differences are in the README (installation commands, config file locations, MSIX gotcha). The SKILL.md itself can be truly cross-platform.

---

## 8. Ready to Build

Phase 1 research is complete. All major questions are answered. We can proceed directly to Phase 2–6 with high confidence. No Windows machine needed for initial build — the path conventions are confirmed from official documentation.
