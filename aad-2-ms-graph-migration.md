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

---

## Background

### Why This Migration is Critical

1. **Azure AD Graph API Deprecation**: Microsoft announced deprecation of `https://graph.windows.net` endpoint
2. **Microsoft Graph is the Future**: All new Azure AD/Entra ID features only available via Microsoft Graph
3. **Customer Impact**: Customers using legacy parameters will need migration path
4. **Compliance**: Must align with Microsoft's platform strategy

### API Comparison

| Aspect | AAD Graph (Legacy) | Microsoft Graph (Modern) |
|--------|-------------------|--------------------------|
| Endpoint | `https://graph.windows.net` | `https://graph.microsoft.com` |
| Audience | `https://graph.windows.net/` | `https://graph.microsoft.com/` |
| Status | ⚠️ Deprecated | ✅ Active Development |
| Parameter Names | `GraphEndpoint`, `GraphAudience` | `MicrosoftGraphUrl`, `MicrosoftGraphEndpointResourceId` |
| Constant Name | `AadGraph` | `MSGraph` |

---

## Current State

### What's Already Migrated ✅

1. **Environment Cmdlet Parameters** (Completed in previous release):
   - `Add-AzEnvironment` now accepts `-MicrosoftGraphUrl` and `-MicrosoftGraphEndpointResourceId`
   - `Set-AzEnvironment` now accepts `-MicrosoftGraphUrl` and `-MicrosoftGraphEndpointResourceId`
   - Documented in `ChangeLog.md` line 5838

2. **Constants Defined**:
   - `MSGraph` constant defined in `SupportedResourceNames.cs` line 24
   - Mapping to `MicrosoftGraphEndpointResourceId` in line 49

3. **MSGraph.Autorest Module**:
   - Full module generated from Microsoft Graph OpenAPI specs
   - Located in `/generated/MSGraph/MSGraph.Autorest/`

### What Needs Migration ⚠️

1. **Legacy Parameters Still Active**:
   - `GraphEndpoint` parameter in Add/Set-AzEnvironment (NO deprecation warning)
   - `GraphAudience` parameter in Add/Set-AzEnvironment (NO deprecation warning)
   - Aliases: `Graph`, `GraphUrl`, `GraphEndpointResourceId`, `GraphResourceId`

2. **Test Infrastructure**:
   - Test environments use AAD Graph PPE: `https://graph.ppe.windows.net/`
   - Default token audience: `https://graph.windows.net/` (line 123 in TestEndpoints.cs)

3. **Authentication Layer**:
   - Dual authentication paths maintained for backward compatibility
   - Both `environment.GraphEndpointResourceId` and `MicrosoftGraphEndpointResourceId` supported

---

## Related GitHub Issues

### Open Issues

**Issue #28209** - [Set-AzEnvironment -SshAuthScope errors for built-in environments](https://github.com/Azure/azure-powershell/issues/28209)
- **Status:** OPEN (Created: 2025-07-17)
- **Labels:** bug, customer-reported, Accounts, Tracking
- **Assignee:** isra-fel
- **Description:** Cannot set SshAuthScope on discovered/built-in environments due to validation blocking modifications
- **Relevance:** Related to environment configuration and discovered vs. custom environment handling
- **Lines Affected:** `src/Accounts/Accounts/Environment/SetAzureRMEnvironment.cs:214-218`

### Closed Issues (Historical Context)

All AAD/Graph-related issues are **CLOSED**:

**Issue #16319** - [New-AzRoleAssignment fails when authenticated as Service Principal](https://github.com/Azure/azure-powershell/issues/16319)
- **Status:** CLOSED (2021-11-02 to 2021-11-03)
- **Error:** Authorization_RequestDenied on `https://graph.windows.net` endpoint
- **Root Cause:** Service principal lacks AAD Graph permissions

**Issue #15635** - [Az.KeyVault 3.4.4 purge protection policy issue](https://github.com/Azure/azure-powershell/issues/15635)
- **Status:** CLOSED (2021-08-10 to 2021-08-31)
- **Relevance:** Shows AAD Graph authentication in debug logs

**Issue #13573** - [Get-AzRoleAssignment produces error when called as SPN](https://github.com/Azure/azure-powershell/issues/13573)
- **Status:** CLOSED (2020-11-02)
- **Error:** CloudException on AAD Graph getObjectsByObjectIds endpoint
- **Root Cause:** Insufficient AAD Graph permissions for service principal

**Issue #13127** - [KeyVaultAccessToken is ignored](https://github.com/Azure/azure-powershell/issues/13127)
- **Status:** CLOSED (2020-10-01 to 2020-11-17, Milestone S178)
- **Error:** Management token used instead of Vault token
- **Shows:** Triple token authentication (management, graph, vault)

**Issue #12840** - [Get-AzRoleAssignment Authorization_RequestDenied for SPN](https://github.com/Azure/azure-powershell/issues/12840)
- **Status:** CLOSED (2020-09-02)
- **Debug Output:** POST to `https://graph.windows.net/.../getObjectsByObjectIds`
- **Error:** "Insufficient privileges to complete the operation"

**Issue #3215** - [New-AzureRmADApplication fails for SPN](https://github.com/Azure/azure-powershell/issues/3215)
- **Status:** OPEN (Created: 2016-11-21, Last Updated: 2021-11-11)
- **Error:** "Resource not found for the segment 'me'" on AAD Graph endpoint
- **Historical:** Shows long-standing AAD Graph permission issues

---

## Code Changes Required

### 1. Environment Cmdlets - Parameter Deprecation

**File:** `/workspaces/azure-powershell/src/Accounts/Accounts/Environment/AddAzureRMEnvironment.cs`

#### Lines 90-98: GraphEndpoint Parameter (DEPRECATE)
```csharp
// CURRENT CODE (NO DEPRECATION WARNING):
[Parameter(Mandatory = false, HelpMessage = "The Active Directory graph endpoint", ValueFromPipelineByPropertyName = true)]
[Alias("Graph", "GraphUrl")]
public string GraphEndpoint { get; set; }

// REQUIRED CHANGE:
[Parameter(Mandatory = false, HelpMessage = "The Active Directory graph endpoint (DEPRECATED - Use MicrosoftGraphUrl instead)", ValueFromPipelineByPropertyName = true)]
[Alias("Graph", "GraphUrl")]
[Obsolete("GraphEndpoint is deprecated. Please use MicrosoftGraphUrl parameter instead. This parameter will be removed in a future release.", false)]
public string GraphEndpoint { get; set; }
```

#### Lines 138: GraphAudience Parameter (DEPRECATE)
```csharp
// CURRENT CODE (NO DEPRECATION WARNING):
[Parameter(Mandatory = false, HelpMessage = "The audience for tokens authenticating with the AD Graph endpoint", ValueFromPipelineByPropertyName = true)]
[Alias("GraphEndpointResourceId", "GraphResourceId")]
public string GraphAudience { get; set; }

// REQUIRED CHANGE:
[Parameter(Mandatory = false, HelpMessage = "The audience for tokens authenticating with the AD Graph endpoint (DEPRECATED - Use MicrosoftGraphEndpointResourceId instead)", ValueFromPipelineByPropertyName = true)]
[Alias("GraphEndpointResourceId", "GraphResourceId")]
[Obsolete("GraphAudience is deprecated. Please use MicrosoftGraphEndpointResourceId parameter instead. This parameter will be removed in a future release.", false)]
public string GraphAudience { get; set; }
```

#### Lines 279-281: Metadata Retrieval Logic (UPDATE)
```csharp
// CURRENT CODE:
environment.GraphEndpoint = metadataEndpoints["graphEndpoint"];
environment.GraphAudience = metadataEndpoints["graph"];

// REQUIRED CHANGE (map legacy to modern):
// Set modern properties from metadata
environment.MicrosoftGraphUrl = metadataEndpoints["graphEndpoint"];
environment.MicrosoftGraphEndpointResourceId = metadataEndpoints["graph"];
// Also set legacy for backward compatibility (temporary)
environment.GraphEndpoint = metadataEndpoints["graphEndpoint"];
environment.GraphAudience = metadataEndpoints["graph"];
```

**File:** `/workspaces/azure-powershell/src/Accounts/Accounts/Environment/SetAzureRMEnvironment.cs`

Apply identical deprecation changes to `Set-AzEnvironment` cmdlet (similar parameter structure).

**Priority:** HIGH  
**Breaking Change:** Phase 1: Soft deprecation (warnings), Phase 2: Parameter removal

---

### 2. Test Framework - Endpoint Updates

**File:** `/workspaces/azure-powershell/tools/TestFx/TestEndpoints.cs`

#### Line 123: Default Graph Token Audience (UPDATE)
```csharp
// CURRENT CODE:
defaultGraphTokenAudienceUri = "https://graph.windows.net/";

// REQUIRED CHANGE:
defaultGraphTokenAudienceUri = "https://graph.microsoft.com/";
```

#### Lines 138, 152, 164, 170: Test Environment Graph URLs (UPDATE)

**Dogfood Environment (Lines 183-234):**
```csharp
// CURRENT CODE (Line ~138):
GraphUri = new Uri("https://graph.ppe.windows.net/")

// REQUIRED CHANGE:
GraphUri = new Uri("https://graph.microsoft-ppe.com/")  // If PPE exists for MS Graph
// OR use production endpoint if PPE not available:
GraphUri = new Uri("https://graph.microsoft.com/")
```

**Next Environment:**
```csharp
// CURRENT CODE (Line ~152):
GraphUri = new Uri("https://graph.ppe.windows.net/")

// REQUIRED CHANGE:
GraphUri = new Uri("https://graph.microsoft.com/")
```

**Current Environment:**
```csharp
// CURRENT CODE (Line ~164):
GraphUri = new Uri("https://graph.ppe.windows.net/")

// REQUIRED CHANGE:
GraphUri = new Uri("https://graph.microsoft.com/")
```

**Custom Test Environment:**
```csharp
// CURRENT CODE (Line ~170):
GraphUri = new Uri("https://graph.ppe.windows.net/")

// REQUIRED CHANGE:
GraphUri = new Uri("https://graph.microsoft.com/")
```

**Priority:** MEDIUM (after cmdlet deprecation)  
**Breaking Change:** No (internal test infrastructure)

---

### 3. Authentication Layer - Dual Support

**File:** `/workspaces/azure-powershell/src/Accounts/Authenticators/AccessTokenAuthenticator.cs`

#### Lines 59-63: Legacy AAD Graph Authentication (PHASE OUT)
```csharp
// CURRENT CODE:
if (!string.IsNullOrEmpty(environment.GraphEndpointResourceId))
{
    tokenCache[environment.GraphEndpointResourceId] = GetAccessToken(
        account, environment, environment.GraphEndpointResourceId, credentialFactory, promptBehavior, promptAction, tenant);
}

// PHASE 1: Keep for backward compatibility, add logging
if (!string.IsNullOrEmpty(environment.GraphEndpointResourceId))
{
    WriteWarningMessage("GraphEndpointResourceId is deprecated. Please use MicrosoftGraphEndpointResourceId.");
    tokenCache[environment.GraphEndpointResourceId] = GetAccessToken(
        account, environment, environment.GraphEndpointResourceId, credentialFactory, promptBehavior, promptAction, tenant);
}

// PHASE 2: Remove after deprecation period
// (Delete this entire block)
```

#### Lines 77-79: Modern MS Graph Authentication (KEEP)
```csharp
// CURRENT CODE (CORRECT - NO CHANGES NEEDED):
if (!string.IsNullOrEmpty(MicrosoftGraphEndpointResourceId))
{
    tokenCache[MicrosoftGraphEndpointResourceId] = GetAccessToken(
        account, environment, MicrosoftGraphEndpointResourceId, credentialFactory, promptBehavior, promptAction, tenant);
}
```

**Priority:** LOW (after cmdlets and tests migrated)  
**Breaking Change:** Phase 1: No, Phase 2: Yes (only affects code using old properties)

---

### 4. Constants - Maintain Both (Temporary)

**File:** `/workspaces/azure-powershell/src/Accounts/Accounts/Utilities/SupportedResourceNames.cs`

#### Lines 23-24: Constants Definition (CURRENT STATE OK)
```csharp
// CURRENT CODE (NO CHANGE NEEDED):
public const string AadGraph = "AadGraph";  // Line 23 - Legacy
public const string MSGraph = "MSGraph";    // Line 24 - Modern
```

#### Lines 48-49: Resource ID Mapping (CURRENT STATE OK)
```csharp
// CURRENT CODE (NO CHANGE NEEDED):
{ AadGraph, AzureEnvironment.Endpoint.GraphEndpointResourceId },     // Line 48 - Legacy
{ MSGraph, AzureEnvironment.Endpoint.MicrosoftGraphEndpointResourceId }, // Line 49 - Modern
```

**Note:** Keep both constants during migration period for backward compatibility. Remove `AadGraph` in Phase 2.

**Priority:** LOW (deprecate after authentication layer migration)  
**Breaking Change:** Phase 2 only

---

## Migration Strategy

### Phase 1: Soft Deprecation (Current Sprint - 3-4 weeks)

**Goal:** Warn customers, maintain backward compatibility

1. **Add Deprecation Warnings**
   - [ ] Add `[Obsolete]` attributes to `GraphEndpoint` and `GraphAudience` parameters
   - [ ] Update parameter help messages with deprecation notice
   - [ ] Add runtime warnings in authentication layer when legacy properties used
   
2. **Update Documentation**
   - [ ] Update cmdlet help files with migration guidance
   - [ ] Update `ChangeLog.md` with deprecation announcement
   - [ ] Create migration guide for customers
   
3. **Communication**
   - [ ] Publish blog post announcing deprecation timeline
   - [ ] Update breaking changes documentation
   - [ ] Send notifications to known affected customers

**Success Criteria:**
- All legacy parameters have deprecation warnings
- Documentation clearly states migration path
- No functional regressions

---

### Phase 2: Test Infrastructure Migration (Sprint + 1-2 weeks)

**Goal:** Migrate test infrastructure to MS Graph

1. **Update Test Endpoints**
   - [ ] Change default graph token audience to MS Graph
   - [ ] Update all test environment graph URLs
   - [ ] Verify test credentials have MS Graph permissions
   
2. **Test Execution**
   - [ ] Run full regression test suite
   - [ ] Fix any test failures due to endpoint changes
   - [ ] Verify recording/playback scenarios work
   
3. **CI/CD Pipeline**
   - [ ] Update build scripts if needed
   - [ ] Verify CloudShell deployment tests pass

**Success Criteria:**
- All tests pass with MS Graph endpoints
- No AAD Graph endpoints in test configurations
- Test credentials validated for MS Graph

---

### Phase 3: Code Cleanup (Sprint + 2-3 months)

**Goal:** Remove legacy code, finalize migration

**Prerequisite:** Minimum 2 releases with deprecation warnings active

1. **Remove Deprecated Parameters** (BREAKING CHANGE)
   - [ ] Remove `GraphEndpoint` parameter from Add-AzEnvironment
   - [ ] Remove `GraphAudience` parameter from Add-AzEnvironment
   - [ ] Remove `GraphEndpoint` parameter from Set-AzEnvironment
   - [ ] Remove `GraphAudience` parameter from Set-AzEnvironment
   - [ ] Remove parameter aliases: `Graph`, `GraphUrl`, `GraphEndpointResourceId`, `GraphResourceId`

2. **Remove Legacy Authentication Code**
   - [ ] Remove AAD Graph authentication path (AccessTokenAuthenticator.cs lines 59-63)
   - [ ] Remove `GraphEndpointResourceId` property references
   - [ ] Keep only `MicrosoftGraphEndpointResourceId` path

3. **Remove Legacy Constants**
   - [ ] Remove `AadGraph` constant from SupportedResourceNames.cs
   - [ ] Remove AAD Graph mapping from resource ID dictionary
   
4. **Final Validation**
   - [ ] Full regression testing
   - [ ] Security review
   - [ ] Performance testing
   - [ ] Breaking changes review

**Success Criteria:**
- No AAD Graph code remains
- All tests pass
- Breaking changes properly documented
- Customer migration guide validated

---

## Breaking Changes

### Phase 1 (Soft Deprecation) - NO BREAKING CHANGES

**Customer Impact:** Warnings only, all functionality preserved

**Action Required:**
- Customers should update scripts to use new parameters
- No immediate action required

**Example Warning:**
```
WARNING: GraphEndpoint parameter is deprecated and will be removed in a future release. 
Please use MicrosoftGraphUrl parameter instead.
```

---

### Phase 2 (Test Infrastructure) - NO CUSTOMER IMPACT

**Customer Impact:** None (internal testing only)

---

### Phase 3 (Code Removal) - BREAKING CHANGES

**Removal Date:** TBD (minimum 2 releases after Phase 1)

#### Removed Parameters:
1. **Add-AzEnvironment**:
   - ❌ `-GraphEndpoint` (use `-MicrosoftGraphUrl`)
   - ❌ `-GraphAudience` (use `-MicrosoftGraphEndpointResourceId`)
   - ❌ Aliases: `-Graph`, `-GraphUrl`, `-GraphEndpointResourceId`, `-GraphResourceId`

2. **Set-AzEnvironment**:
   - ❌ `-GraphEndpoint` (use `-MicrosoftGraphUrl`)
   - ❌ `-GraphAudience` (use `-MicrosoftGraphEndpointResourceId`)
   - ❌ Aliases: `-Graph`, `-GraphUrl`, `-GraphEndpointResourceId`, `-GraphResourceId`

#### Migration Examples:

**Before (Legacy - Will Break):**
```powershell
Add-AzEnvironment -Name "CustomCloud" `
    -GraphEndpoint "https://graph.windows.net/" `
    -GraphAudience "https://graph.windows.net/"
```

**After (Modern - Required):**
```powershell
Add-AzEnvironment -Name "CustomCloud" `
    -MicrosoftGraphUrl "https://graph.microsoft.com/" `
    -MicrosoftGraphEndpointResourceId "https://graph.microsoft.com/"
```

#### Impacted Scenarios:
1. **Custom Cloud Environments**: Customers defining custom Azure clouds with Graph endpoints
2. **Automation Scripts**: Scripts explicitly setting Graph parameters
3. **Discovered Environments**: Environments auto-discovered via metadata (should auto-migrate)

#### Mitigation:
- Provide script migration tool to automatically update customer scripts
- Maintain comprehensive migration documentation
- Offer support during transition period

---

## Testing Requirements

### Unit Tests

**New Tests Required:**

1. **Deprecation Warning Tests**:
   ```csharp
   // File: src/Accounts/Accounts.Test/EnvironmentCmdletTests.cs
   [Fact]
   public void AddAzEnvironment_WithGraphEndpoint_ShowsDeprecationWarning()
   {
       // Test that using -GraphEndpoint shows Obsolete warning
   }
   
   [Fact]
   public void AddAzEnvironment_WithGraphAudience_ShowsDeprecationWarning()
   {
       // Test that using -GraphAudience shows Obsolete warning
   }
   ```

2. **Modern Parameter Tests**:
   ```csharp
   [Fact]
   public void AddAzEnvironment_WithMicrosoftGraphUrl_Succeeds()
   {
       // Test modern parameter works correctly
   }
   
   [Fact]
   public void AddAzEnvironment_WithBothOldAndNew_PrefersMSGraph()
   {
       // Test backward compatibility if both provided
   }
   ```

3. **Authentication Token Tests**:
   ```csharp
   [Fact]
   public void AccessTokenAuthenticator_UsesMSGraphEndpoint()
   {
       // Verify token acquired for MS Graph endpoint
   }
   ```

---

### Integration Tests

1. **Environment Creation**:
   - Create custom environment with MS Graph parameters
   - Verify environment persists correctly
   - Test authentication against custom environment

2. **Token Acquisition**:
   - Authenticate to custom environment
   - Verify MS Graph token acquired
   - Test cmdlets using MS Graph permissions

3. **Regression Tests**:
   - Run full Az.Accounts test suite
   - Run Az.Resources test suite (AAD cmdlets)
   - Verify no functionality regressions

---

### Manual Testing Scenarios

1. **Add Custom Environment with MS Graph**:
   ```powershell
   Add-AzEnvironment -Name "TestCloud" `
       -ActiveDirectoryEndpoint "https://login.test.com" `
       -MicrosoftGraphUrl "https://graph.microsoft.com/" `
       -MicrosoftGraphEndpointResourceId "https://graph.microsoft.com/" `
       -ResourceManagerEndpoint "https://management.test.com"
   
   Connect-AzAccount -Environment "TestCloud"
   Get-AzADUser  # Should work with MS Graph
   ```

2. **Legacy Parameter Warning**:
   ```powershell
   Add-AzEnvironment -Name "LegacyCloud" `
       -GraphEndpoint "https://graph.windows.net/" `
       -GraphAudience "https://graph.windows.net/"
   
   # Expected: Deprecation warning shown
   ```

3. **Migration Path**:
   ```powershell
   # Customer has existing environment with legacy parameters
   Get-AzEnvironment -Name "OldCustomCloud"
   
   # Update to MS Graph
   Set-AzEnvironment -Name "OldCustomCloud" `
       -MicrosoftGraphUrl "https://graph.microsoft.com/" `
       -MicrosoftGraphEndpointResourceId "https://graph.microsoft.com/"
   ```

---

## Timeline & Phases

| Phase | Duration | Target Release | Breaking Changes |
|-------|----------|----------------|------------------|
| **Phase 1: Soft Deprecation** | 3-4 weeks | Next Release | ❌ No |
| **Phase 2: Test Migration** | 1-2 weeks | Next + 1 | ❌ No |
| **Phase 3: Code Cleanup** | 2-3 months | Next + 2-3 | ✅ Yes |

**Total Timeline:** ~4-6 months from start to Phase 3 release

**Key Milestones:**
- ✅ MS Graph parameters added (COMPLETED)
- 🔄 Deprecation warnings added (CURRENT PHASE)
- ⏳ Test infrastructure migrated (NEXT)
- ⏳ Legacy code removed (FUTURE)

---

## Communication Plan

### Internal Stakeholders
- [ ] PM team approval for breaking changes
- [ ] Security review for authentication changes
- [ ] CloudShell team notification (test endpoint changes)
- [ ] Documentation team for help updates

### External Communication
- [ ] **Release Notes**: Deprecation announcement in ChangeLog.md
- [ ] **Blog Post**: "Migrating from AAD Graph to Microsoft Graph in Az PowerShell"
- [ ] **Breaking Changes Doc**: Update with Phase 3 removals
- [ ] **Migration Guide**: Step-by-step customer migration instructions

---

## Success Criteria

### Phase 1 Complete When:
- ✅ All legacy parameters have `[Obsolete]` attributes
- ✅ Deprecation warnings show at runtime
- ✅ Documentation updated with migration guidance
- ✅ No functional regressions

### Phase 2 Complete When:
- ✅ Test infrastructure uses MS Graph endpoints exclusively
- ✅ All tests pass with MS Graph
- ✅ No AAD Graph PPE references remain

### Phase 3 Complete When:
- ✅ All legacy AAD Graph code removed
- ✅ Breaking changes fully documented
- ✅ Customer migration guide published
- ✅ Security and compliance reviews passed

---

## Risk Mitigation

### Risk: Customer Scripts Break
**Mitigation:**
- Long deprecation period (2+ releases)
- Clear warnings and documentation
- Automated script migration tool

### Risk: Test Failures
**Mitigation:**
- Thorough regression testing
- Gradual rollout to test environments
- Rollback plan if issues detected

### Risk: Authentication Issues
**Mitigation:**
- Maintain dual authentication during transition
- Extensive integration testing
- Monitor telemetry for auth failures

---

## Appendix

### Useful Resources

1. **Microsoft Graph Documentation**:
   - [Microsoft Graph Overview](https://learn.microsoft.com/graph/overview)
   - [Migrate from Azure AD Graph](https://learn.microsoft.com/graph/migrate-azure-ad-graph-overview)

2. **Azure PowerShell Documentation**:
   - [Development Docs](../documentation/development-docs/)
   - [Breaking Changes Guide](../documentation/breaking-changes/)

3. **Related Repositories**:
   - [Azure SDK for .NET](https://github.com/Azure/azure-sdk-for-net)
   - [Microsoft Graph SDK](https://github.com/microsoftgraph/msgraph-sdk-dotnet)

### Contact Information

**Module Owner:** Az.Accounts Team  
**Technical Lead:** TBD  
**PM Contact:** TBD  

---

**Document Status:** DRAFT - Ready for Team Review  
**Next Review Date:** TBD

