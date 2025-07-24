```
<#
.SYNOPSIS
    Disable Windows PowerShell 2.0 feature on Windows 10, per STIG WN10-00-000155.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-00-000155

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# Function to check for elevation
function Assert-Administrator {
    if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()
        ).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
        Write-Error "ERROR: This script must be run as Administrator."
        exit 1
    }
}

# Function to disable a given optional feature if it's enabled
function Disable-FeatureIfEnabled {
    param(
        [Parameter(Mandatory)][string] $FeatureName
    )
    $feature = Get-WindowsOptionalFeature -Online -FeatureName $FeatureName -ErrorAction Stop
    if ($feature.State -eq 'Enabled' -or $feature.State -eq 'EnabledPending') {
        Write-Output "Disabling feature: $FeatureName ..."
        Disable-WindowsOptionalFeature -Online -FeatureName $FeatureName -NoRestart -ErrorAction Stop |
            ForEach-Object { Write-Output "  $_" }
    } else {
        Write-Output "Feature already disabled: $FeatureName"
    }
}

# MAIN
Assert-Administrator

Write-Output "== STIG WN10-00-000155: Disable PowerShell 2.0 =="

# Disable the two PS 2.0 components
Disable-FeatureIfEnabled -FeatureName 'MicrosoftWindowsPowerShellV2Root'
Disable-FeatureIfEnabled -FeatureName 'MicrosoftWindowsPowerShellV2'

# Verify
Write-Output "`nVerification:"
Get-WindowsOptionalFeature -Online `
  -FeatureName 'MicrosoftWindowsPowerShellV2Root','MicrosoftWindowsPowerShellV2' |
  Format-Table FeatureName, State -AutoSize

# Ask to reboot if any feature was disabled
$needsReboot = (Get-WindowsOptionalFeature -Online `
    -FeatureName 'MicrosoftWindowsPowerShellV2Root','MicrosoftWindowsPowerShellV2' |
    Where-Object State -Match 'DisabledPending').Count -gt 0

if ($needsReboot) {
    Write-Warning "One or more features require a reboot to complete the disablement."
    $resp = Read-Host "Reboot now? (Y/N)"
    if ($resp -match '^[Yy]') {
        Write-Output "Rebooting..."
        Restart-Computer
    } else {
        Write-Output "Remember to reboot later to finalize changes."
    }
} else {
    Write-Output "No reboot is required."
}

Write-Output "Done."
```