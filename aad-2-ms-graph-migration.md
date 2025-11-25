# AAD Graph to Microsoft Graph API Migration

**Last Updated:** November 25, 2025

## Code That Still Needs Migration

### 1. Environment Cmdlet Parameters - Add Deprecation Warnings

**File:** `src/Accounts/Accounts/Environment/AddAzureRMEnvironment.cs`
- **Lines 90-98:** `GraphEndpoint` parameter - needs `[Obsolete]` attribute
- **Line 138:** `GraphAudience` parameter - needs `[Obsolete]` attribute  
- **Lines 279-281:** Metadata retrieval - sets legacy Graph properties

**File:** `src/Accounts/Accounts/Environment/SetAzureRMEnvironment.cs`
- Same parameters as Add-AzEnvironment need deprecation warnings

### 2. Test Framework - Update Endpoints

**File:** `tools/TestFx/TestEndpoints.cs`
- **Line 123:** Default audience `"https://graph.windows.net/"` → change to `"https://graph.microsoft.com/"`
- **Line 138:** Dogfood environment uses `"https://graph.ppe.windows.net/"` → change to MS Graph
- **Line 152:** Next environment uses `"https://graph.ppe.windows.net/"` → change to MS Graph
- **Line 164:** Current environment uses `"https://graph.ppe.windows.net/"` → change to MS Graph
- **Line 170:** Custom environment uses `"https://graph.ppe.windows.net/"` → change to MS Graph

### 3. Authentication Layer - Remove Legacy Support

**File:** `src/Accounts/Authenticators/AccessTokenAuthenticator.cs`
- **Lines 59-63:** Legacy AAD Graph authentication path - remove after deprecation period
- **Lines 77-79:** MS Graph authentication - keep (already correct)

### 4. Constants - Remove Legacy (Final Phase)

**File:** `src/Accounts/Accounts/Utilities/SupportedResourceNames.cs`
- **Line 23:** `AadGraph` constant - remove in final phase
- **Line 48:** AAD Graph mapping - remove in final phase

---

## Open GitHub Issues Related to AAD/MS Graph Migration

### Issue #28209 - [Set-AzEnvironment -SshAuthScope errors](https://github.com/Azure/azure-powershell/issues/28209)
- **Status:** OPEN  
- **Created:** July 17, 2025
- **Assignee:** isra-fel
- **Problem:** Cannot set SshAuthScope on built-in/discovered environments
- **Related To:** Environment configuration validation blocking modifications
- **File:** `src/Accounts/Accounts/Environment/SetAzureRMEnvironment.cs:214-218`

### No Other Open Issues

All other AAD Graph/MS Graph related issues are **CLOSED**:
- #16319 - Service principal AAD Graph permissions (closed Nov 2021)
- #15635 - KeyVault AAD Graph auth (closed Aug 2021)  
- #13573 - SPN role assignment AAD Graph error (closed Nov 2020)
- #13127 - KeyVault token ignored (closed Nov 2020)
- #12840 - Get-AzRoleAssignment SPN error (closed Sep 2020)
- #3215 - New-AzureRmADApplication SPN error (open since Nov 2016, but inactive)
