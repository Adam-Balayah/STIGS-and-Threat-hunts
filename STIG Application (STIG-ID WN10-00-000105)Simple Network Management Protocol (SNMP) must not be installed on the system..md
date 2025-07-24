```
<#
.SYNOPSIS
    Uninstalls Simple Network Management Protocol (SNMP) from a Windows 10 system.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WIN10-SO-000245

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

.DESCRIPTION
    Per DISA STIG WN10-00-000105, SNMP must not be installed on workstations.
    This script checks for the SNMP-Service and SNMP-WMI-Provider Windows features
    and, if present, disables and removes them via DISM.


#>

# --- 1) Elevation check ---
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()
    ).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Error "ERROR: This script must be run as Administrator."
    exit 1
}

# --- 2) Define the SNMP-related features ---
$snmpFeatures = @(
    "SNMP-Service",        # Core SNMP service
    "SNMP-WMI-Provider"    # WMI provider for SNMP
)

foreach ($feature in $snmpFeatures) {
    Write-Host "Checking for feature '$feature'..." -ForegroundColor Cyan

    # Query the current state of the feature
    $info = dism.exe /Online /Get-FeatureInfo /FeatureName:$feature 2>&1
    if ($LASTEXITCODE -ne 0) {
        Write-Warning "  • Feature '$feature' not found or cannot be queried. Skipping."
        continue
    }

    # Parse out the State line (e.g. "State : Enabled")
    if ($info -match 'State\s*:\s*(\w+)') {
        $state = $matches[1]
    } else {
        Write-Warning "  • Unexpected output for '$feature'. Skipping."
        continue
    }

    # If the feature is enabled or pending enablement, remove it
    if ($state -ieq "Enabled" -or $state -ieq "EnablePending") {
        Write-Host "  • Feature is $state. Disabling and removing payload..." -ForegroundColor Yellow

        $removeOutput = dism.exe /Online /Disable-Feature /FeatureName:$feature /Remove /NoRestart 2>&1
        if ($LASTEXITCODE -eq 0) {
            Write-Host "  ✓ Successfully removed '$feature'." -ForegroundColor Green
        } else {
            Write-Error "  ✗ Failed to remove '$feature'. Details:`n$removeOutput"
        }
    }
    else {
        Write-Host "  • Feature is $state. No action needed." -ForegroundColor Green
    }
}

Write-Host "`nAll done. A reboot is recommended to complete removal of any pending components." -ForegroundColor Cyan


```