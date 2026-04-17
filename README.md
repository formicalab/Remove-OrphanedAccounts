# Remove-OrphanedAccounts

PowerShell 7 script to find and remove orphaned RBAC role assignments in Azure subscriptions and management groups.

## Overview

When users, groups, or service principals are deleted from Microsoft Entra ID (Azure AD), their RBAC role assignments in Azure may remain as "orphaned" assignments. These orphaned assignments:

- Reference deleted principals that can no longer be resolved
- Clutter the IAM view in Azure portal
- May pose security audit concerns

This script scans all accessible Azure subscriptions (and optionally management groups) to identify and remove these orphaned role assignments.

## Features

- ✅ Scans all subscriptions in the tenant
- ✅ Detects orphaned assignments at subscription, resource group, and resource level
- ✅ Optionally scans management groups
- ✅ Interactive deletion with confirmation prompts
- ✅ Supports `-WhatIf` mode for safe preview
- ✅ Supports `-Force` mode for automated cleanup
- ✅ Export findings to CSV with `-ExportPath`
- ✅ Scan-only mode with `-ScanOnly` (no deletion prompts)
- ✅ Progress indicators for long-running scans
- ✅ Actionable error guidance for common issues
- ✅ PowerShell 7 native features (strict mode, proper error handling)

## Requirements

- PowerShell 7.0 or later
- Az PowerShell modules:
  - `Az.Accounts`
  - `Az.Resources`
- Azure permissions: Must have `Microsoft.Authorization/roleAssignments/read` and `Microsoft.Authorization/roleAssignments/delete` permissions on the scopes being scanned

## Installation

1. Clone this repository or download the script
2. Ensure you have the required Az modules installed:

```powershell
Install-Module -Name Az.Accounts -Scope CurrentUser
Install-Module -Name Az.Resources -Scope CurrentUser
```

## Usage

### Basic Usage

First, log in to Azure:

```powershell
Connect-AzAccount
```

Then run the script:

```powershell
# Preview mode - see what would be deleted without making changes
.\remove-orphanedaccounts.ps1 -WhatIf

# Interactive mode - confirm each deletion
.\remove-orphanedaccounts.ps1

# Automatic removal without prompts
.\remove-orphanedaccounts.ps1 -Force
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `-SubscriptionId` | Optional. Specify one or more subscription IDs to scan. If not provided, all accessible subscriptions are scanned. |
| `-IncludeManagementGroups` | Also scan management groups for orphaned role assignments. |
| `-Force` | Skip confirmation prompts and remove all orphaned assignments automatically. |
| `-ExportPath` | Export the list of orphaned assignments to a CSV file at the specified path. |
| `-ScanOnly` | Only scan and report orphaned assignments without prompting for removal. |
| `-WhatIf` | Preview what would be removed without making any changes. |
| `-Verbose` | Show detailed progress and diagnostic information. |

### Examples

```powershell
# Scan all subscriptions (preview mode)
.\remove-orphanedaccounts.ps1 -WhatIf

# Scan a specific subscription
.\remove-orphanedaccounts.ps1 -SubscriptionId "12345678-1234-1234-1234-123456789abc"

# Scan subscriptions and management groups
.\remove-orphanedaccounts.ps1 -IncludeManagementGroups -WhatIf

# Scan only and export to CSV (no removal prompts)
.\remove-orphanedaccounts.ps1 -ScanOnly -ExportPath ".\orphaned-assignments.csv"

# Scan management groups and export findings
.\remove-orphanedaccounts.ps1 -IncludeManagementGroups -ScanOnly -ExportPath ".\audit-report.csv"

# Remove all orphaned assignments without prompting
.\remove-orphanedaccounts.ps1 -Force

# Verbose output for troubleshooting
.\remove-orphanedaccounts.ps1 -Verbose -WhatIf
```

### Interactive Prompts

When running in interactive mode (without `-Force`), you'll be prompted for each orphaned assignment:

- `[Y]` Yes - Remove this assignment
- `[N]` No - Skip this assignment
- `[A]` Yes to All - Remove all remaining assignments without prompting
- `[S]` Skip All - Stop processing and skip all remaining assignments

## How It Works

1. **Validates Azure connection** - Ensures you're logged in to Azure
2. **Retrieves subscriptions** - Gets all accessible subscriptions (or specified ones)
3. **Scans for orphaned assignments** - For each subscription, queries role assignments and identifies orphans based on:
   - `ObjectType` = 'Unknown'
   - Empty `DisplayName`
   - `DisplayName` equals `ObjectId` (fallback display)
4. **Displays findings** - Shows all orphaned assignments grouped by subscription
5. **Exports (optional)** - Saves findings to CSV if `-ExportPath` is specified
6. **Processes deletions** - Prompts for confirmation (unless `-Force`, `-WhatIf`, or `-ScanOnly`)

## Troubleshooting

### Management Group Error: "Value cannot be null. (Parameter 'g')"

This error when using `-IncludeManagementGroups` typically indicates:

1. **Missing permissions** - You need "Management Group Reader" role at the tenant root
2. **Az.Resources module issue** - Update the module: `Update-Module Az.Resources -Force`
3. **Tenant context issue** - Re-authenticate with explicit tenant: `Connect-AzAccount -TenantId <your-tenant-id>`

The script will continue scanning subscriptions even if management group access fails.

### Authorization Errors

Ensure you have the required RBAC permissions:
- `Microsoft.Authorization/roleAssignments/read` - To scan for orphaned assignments
- `Microsoft.Authorization/roleAssignments/delete` - To remove orphaned assignments
- `Microsoft.Management/managementGroups/read` - To scan management groups (optional)

## License

MIT License - See [LICENSE](LICENSE) for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
