# Privilege Cloud Operations

## Create User (API)

```python
# From Identity iframe context (antixss cookie required)
# POST /CDirectoryService/CreateUser
# Username MUST be in format: name@cyberark.cloud.<TENANT_NUM>
# Plain name without domain suffix returns: "user names must be of the form 'name@suffix'"
# UUID returned directly in CreateUser response as d.Result.UUID
```

## Create Safe (API)

```python
# From pcloud tab context
# POST /PasswordVault/api/Safes
# DO NOT set managingCPM unless you know the exact CPM name — causes 400 error
# Query available CPM: GET /PasswordVault/api/ComponentsMonitoringSummary
# Must be called from pcloud domain context, not Angular shell
```

## Add User to Safe — CRITICAL FLOW

**Do NOT skip the role assignment step.** Adding a user to a safe will fail with "user cannot be added" if the user doesn't have the **Privilege Cloud Users** role in Identity.

**Step 1:** Assign "Privilege Cloud Users" role first:
```python
# From Identity iframe context
# POST /SaasManage/AddUsersAndGroupsToRole
# Name: "Privilege_Cloud_Users_ID" (exact string, NOT display name)
# Users: ["<user-uuid>"] (internal UUID, not login name)
```

**Step 2:** Add user to safe with permissions:
```python
# From pcloud tab context
# POST /PasswordVault/api/Safes/<safe-name>/Members
# Permission presets:
#   Full = all 20 fields true
#   Read only = useAccounts, retrieveAccounts, listAccounts
#   Approver = see portal for exact fields
```

## Platform Management

All platform API calls use `X-XSRF-TOKEN` header from pcloud tab context.

**List platforms:** `GET /PasswordVault/api/Platforms`

**Duplicate a platform:**
```
POST /PasswordVault/api/Platforms/<sourceID>/duplicate
Body: {PlatformName: "...", Description: "..."}
```
- Parameter is `PlatformName` (NOT `Name`) — wrong key returns PASWS103E
- sourceID = string ID (e.g. "WinDomain"), NOT numeric
- Endpoint `/api/Platforms/Targets/<id>/Duplicate` (capital T) expects numeric ID → fails — do NOT use
- Correct: `/api/Platforms/<id>/duplicate` (lowercase d, no "Targets" prefix)

**Activate/Deactivate:** `POST /PasswordVault/api/Platforms/<platformID>/activate` or `/deactivate`

**Delete:** `DELETE /PasswordVault/api/Platforms/<platformID>` (only works on inactive platforms)

**Edit platform password generation settings (Export → modify → Import):**

CyberArk REST API has NO PUT/PATCH for platform content. Only way: Export → edit INI → Import.

- Endpoint: `POST /PasswordVault/api/Platforms/import` — **lowercase `i`** (NOT `/Import`)
- Content-Type: `application/json` — NOT multipart/form-data
- Body: `{"ImportFile": "<base64-encoded ZIP string>"}`
- Platform must NOT already exist — delete it first (deactivate + delete), then import

**Why `/Import` (uppercase) fails:** UI uses lowercase `/import` with JSON body. Uppercase variant expects multipart/form-data but rejects all field names with PASWS168E.

**INI parameters for password generation:**

| Parameter | Description | Typical value |
|---|---|---|
| `PasswordLength` | Total password length | 32 |
| `MinUpperCase` | Min uppercase letters | 4 |
| `MinLowerCase` | Min lowercase letters | 4 |
| `MinDigit` | Min digits | 4 |
| `MinSpecial` | Min special characters | 4 |

**Full platform edit flow:**
1. `POST /api/Platforms/<id>/Export` → ZIP
2. Extract ZIP → modify INI → repack ZIP
3. `POST /api/Platforms/<id>/deactivate` → 200
4. `DELETE /api/Platforms/<id>` → 204
5. `POST /api/Platforms/import {"ImportFile": "<base64 ZIP>"}` → 201
6. `POST /api/Platforms/<id>/activate` → 200
