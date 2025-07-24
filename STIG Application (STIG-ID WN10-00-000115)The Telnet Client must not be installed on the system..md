```
<#
.SYNOPSIS
    Enforce STIG WN10-00-000115: Ensure Telnet Client is not installed on Windows 10.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-00-000115

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>


#region --- Helper: Ensure running as Administrator ---
function Assert-Administrator {
    $isAdmin = ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()
               ).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
    if (-not $isAdmin) {
        Write-Error "ERROR: This script must be run as Administrator."
        exit 1
    }
}
#endregion

#region --- Main ---
Assert-Administrator

try {
    # Query TelnetClient feature state
    $telnetFeature = Get-WindowsOptionalFeature -Online -FeatureName "TelnetClient" -ErrorAction Stop

    switch ($telnetFeature.State) {
        "Enabled" {
            Write-Host "[INFO] Telnet Client is currently INSTALLED. Removing..." -ForegroundColor Yellow
            Disable-WindowsOptionalFeature `
                -Online `
                -FeatureName "TelnetClient" `
                -Remove `
                -NoRestart `
                -ErrorAction Stop

            Write-Host "[SUCCESS] Telnet Client feature has been removed." -ForegroundColor Green
            exit 0
        }
        "Disabled" {
            Write-Host "[OK] Telnet Client is already not installed. No action required." -ForegroundColor Green
            exit 0
        }
        default {
            Write-Warning "[WARN] Unexpected feature state: $($telnetFeature.State). Attempting removal..."
            Disable-WindowsOptionalFeature `
                -Online `
                -FeatureName "TelnetClient" `
                -Remove `
                -NoRestart `
                -ErrorAction Stop

            Write-Host "[SUCCESS] Removal attempted; please verify manually if needed." -ForegroundColor Green
            exit 0
        }
    }
}
catch {
    Write-Error "[ERROR] Failed to enforce STIG WN10-00-000115: $_"
    exit 1
}
#endregion


```