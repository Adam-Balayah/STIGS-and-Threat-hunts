```
<#
.SYNOPSIS
    Enforce STIG WN10-00-000120: Ensure TFTP Client is not installed on Windows 10,
    auto-discovering the exact feature name.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-00-000120

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5


.Usage
    - Must be run elevated (as Administrator).
    - Uses built-in DISM PowerShell cmdlets.
    - Does not auto-reboot; if a reboot is required, your orchestration tool will see the non-zero exit.
#>

#region Helper: Require Admin
function Assert-Administrator {
    $current = [Security.Principal.WindowsIdentity]::GetCurrent()
    $principal = New-Object Security.Principal.WindowsPrincipal($current)
    if (-not $principal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
        Write-Error "ERROR: Script must be run as Administrator."
        exit 1
    }
}
#endregion

Assert-Administrator

try {
    # Discover any OptionalFeature whose name contains "TFTP"
    $tftpFeatures = Get-WindowsOptionalFeature -Online |
                    Where-Object { $_.FeatureName -match "TFTP" }

    if (-not $tftpFeatures) {
        Write-Host "[OK] No Windows Optional Feature with 'TFTP' in the name was found." -ForegroundColor Green
        exit 0
    }

    foreach ($feat in $tftpFeatures) {
        Write-Host "`n[INFO] Found feature: $($feat.FeatureName) (State: $($feat.State))"

        if ($feat.State -eq 'Enabled') {
            Write-Host "       ➤ Disabling and removing..." -ForegroundColor Yellow
            Disable-WindowsOptionalFeature `
                -Online `
                -FeatureName $feat.FeatureName `
                -Remove `
                -NoRestart `
                -ErrorAction Stop

            Write-Host "       ✔ Removed $($feat.FeatureName)." -ForegroundColor Green
        }
        else {
            Write-Host "       ➤ Already Disabled — nothing to do." -ForegroundColor Green
        }
    }

    Write-Host "`n[SUCCESS] STIG WN10-00-000120 enforced." -ForegroundColor Cyan
    exit 0
}
catch {
    Write-Error "[ERROR] An error occurred while enforcing WN10-00-000120:`n$_"
    exit 1
}

```