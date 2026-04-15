# Identity Portal Administration

## Tenant URL Patterns
- Identity Admin: `https://<tenant-id>.id.cyberark.cloud`
- Privilege Cloud portal: `https://<tenant-name>.cyberark.cloud/privilegecloud`
- PAM API base: `https://<tenant-name>.privilegecloud.cyberark.cloud/PasswordVault/API/`
- SCIM API base: `https://<tenant-id>.id.cyberark.cloud/scim/`

## Login Flow

### Option A — Browser UI
1. Open portal URL in browser
2. Enter `username@tenant-id` → Next
3. Enter password
4. MFA: select "Email" → "Send me an email"
5. Read MFA code from email
6. Enter 8-digit code → Authenticate

### Option B — REST API Login (recommended for automation)

The CyberArk login page is a React SPA that blocks standard browser automation (password field resets on blur, button disables on every interaction). Bypass via auth API:

1. `POST /Security/StartAuthentication` → get session ID + mechanism IDs (password + email OTP)
2. `POST /Security/AdvanceAuthentication` with password → expect "StartNextChallenge"
3. `POST /Security/AdvanceAuthentication` with `Action: 'StartOOB'` → expect "OobPending"
4. Read OTP from email
5. `POST /Security/AdvanceAuthentication` with OTP → get JWT, expect "LoginSuccess"
6. Navigate browser to portal with JWT to set session cookies

**Important notes:**
- After navigating with JWT, session cookies are set on ALL sub-domains
- `antixss` cookie (Identity) and `XSRF-TOKEN-*` cookie (pcloud) become available
- Email MFA may NOT appear in UI dropdown but IS available via API
- Session stays valid for hours; if redirected to login, repeat steps

## Portal Architecture (Critical)

The Identity admin portal (`/idadmin/`) is heavily iframe-based:
- Main shell: Angular app at `https://<tenant-id>.id.cyberark.cloud`
- Content panels: loaded in cross-origin iframes
- URL bar does NOT update when navigating sections — use sidebar as source of truth
- Many panels have nested iframes

**Browser automation:** Use `frame="iframe[src*='keyword']"` parameter to access iframe content. For nested: `frame="iframe[src*='outer'] >> iframe[src*='inner']"`

## Navigation Reference

| Section | Sidebar Path | iframe keyword |
|---------|-------------|----------------|
| Users | Core Services → Users | `idadmin` |
| Roles | Core Services → Roles | `idadmin` |
| Policies | Core Services → Policies | `idadmin` |
| Web Apps | Apps & Widgets → Web Apps | `idadmin` |
| Auth Profiles | Settings → Authentication | `idadmin` |
| Security | Settings → Security | `idadmin` |
| Endpoints | Settings → Endpoints | `idadmin` |
| Reports | Reports | `idadmin` |
| Privilege Cloud Accounts | Accounts → Accounts View | `pcloud` or `PasswordVault` |

## Common Admin Tasks

**List all users:** Core Services → Users → All Users filter. Columns: Login Name, Display Name, Source Directory, Status, Risk Level, Last Login.

**Open user details:** Click username → tabs: Profile, Account, Groups, Roles, Activity, Administrative Rights.

**List roles:** Core Services → Roles. Columns: Name, Type (Static/Dynamic), Description.

**Open a policy:** Core Services → Policies → click policy name. Sections: Authentication Policies, Session Parameters, Application Policies, Workforce Password Management, User Security Policies.

**List web apps:** Apps & Widgets → Web Apps. Filter by type (SAML, OIDC, Portal, etc.).

**Check auth profiles:** Settings → Authentication → Authentication Profiles.

## Key Gotchas

- **`--` values in policies** = inherited/default, not explicitly set — trace hierarchy to find effective value
- **Views dropdown** in Accounts View blocks table content — click elsewhere to close it first
- **Save button** — changes are NOT auto-saved; always click Save
- **URL doesn't change** when navigating — use sidebar breadcrumbs, not URL
- **Cross-origin iframes** — JavaScript `evaluate` fails with SecurityError; must use frame locator or CDP
- **Policy order matters** — priority-ordered (top = highest priority); Default Policy is catch-all
- **Role naming** — `APP-` prefix = application access, `AUTH-` prefix = authentication policy binding
- **After role assignment** — changes may take a moment to propagate; user may need to re-login
