# [KB-002] Troubleshooting Log: Newly Created Local AD User Not Syncing to Microsoft 365

## 1. Symptom
After provisioning a new virtual employee account for the Sales department in the local Windows Server Active Directory, the user did not appear in the Microsoft 365 Admin Center after waiting over 30 minutes, preventing Exchange mailbox license assignment.

## 2. Troubleshooting Steps
Followed a systematic end-to-end diagnostic approach (Local Source -> Sync Engine -> Cloud Tenant):

1. **Verified Local AD User Attributes**:
   * Opened **Active Directory Users and Computers (ADUC)** and inspected the user's `UserPrincipalName (UPN)`. Discovered the UPN suffix was left as the non-routable local domain (`@homelab.local`) instead of the verified custom domain.
2. **Checked Microsoft Entra Connect Synchronization Logs**:
   * Logged into the server hosting **Microsoft Entra Connect** and launched the **Synchronization Service Manager**.
   * Reviewed recent `Delta Synchronization` operations and noticed the newly created departmental Organizational Unit (OU) was excluded from the sync scope.
3. **Audited Sync Filtering Rules**:
   * Opened the Entra Connect configuration wizard and checked **Domain and OU Filtering**, confirming the new Sales sub-OU was unchecked.

## 3. Root Cause
1. **Sync Scope Exclusion**: The newly created departmental OU was not selected in Entra Connect's synchronization filtering settings.
2. **Non-Routable UPN Suffix**: Users without a verified custom domain UPN suffix default to `.onmicrosoft.com` upon syncing, causing mismatches with corporate email routing policies.

## 4. Resolution
1. **Updated UPN Suffix**: Modified the user's UPN suffix in local ADUC to match the verified corporate domain.
2. **Updated Sync Scope**: Selected the new departmental OU within the Entra Connect configuration wizard.
3. **Forced Manual Delta Sync**: Executed the following PowerShell command as Administrator on the sync server to trigger an immediate sync cycle without waiting for the default 30-minute scheduler:
   ```powershell
   Start-ADSyncSyncCycle -PolicyType Delta
   ```
4. **Verification**: Confirmed the user appeared in the Microsoft 365 Admin Center within 2 minutes and successfully assigned the Microsoft 365 license and mailbox.
