```
 <#
.SYNOPSIS
    Uninstalls Internet Information Services (IIS) and Hostable Web Core from a Windows 10 workstation.


.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000075

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : Windows-VM
    PowerShell Ver. : 5
#>

# --- Elevation check ---
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()
    ).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Error "ERROR: This script must be run as Administrator."
    exit 1
}

# List of IIS-related Windows Features to remove
$featuresToRemove = @(
    "IIS-WebServerRole",        # Main IIS role
    "IIS-HostableWebCore"       # Hostable Web Core subcomponent
)

foreach ($feature in $featuresToRemove) {
    Write-Host "Checking feature '$feature'..." -ForegroundColor Cyan

    # Get current feature state
    $info = & dism.exe /Online /Get-FeatureInfo /FeatureName:$feature 2>&1
    if ($LASTEXITCODE -ne 0) {
        Write-Warning "Could not query feature $feature. It may not exist on this OS. Skipping."
        continue
    }

    # Parse state
    if ($info -match "State :\s*(\w+)") {
        $state = $matches[1]
    } else {
        Write-Warning "Unexpected output querying $feature. Skipping."
        continue
    }

    if ($state -ieq "Enabled" -or $state -ieq "EnabledPending") {
        Write-Host "Feature '$feature' is $state. Disabling and removing..." -ForegroundColor Yellow

        # Disable and remove the feature payload
        $removeOutput = & dism.exe /Online /Disable-Feature /FeatureName:$feature /Remove /NoRestart 2>&1
        if ($LASTEXITCODE -eq 0) {
            Write-Host "Successfully disabled and removed '$feature'." -ForegroundColor Green
        } else {
            Write-Error "Failed to remove '$feature'. Details:`n$removeOutput"
        }
    }
    else {
        Write-Host "Feature '$feature' is $state. No action needed." -ForegroundColor Green
    }
}

Write-Host "`nAll done. You may reboot to finalize removal of any pending component changes." -ForegroundColor Cyan

```