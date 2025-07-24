```
<#
.SYNOPSIS
  Enforce DISA STIG WN10-AC-000035: Minimum password length = 14

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-AC-000035

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

.DESCRIPTION
  • Uses `net accounts` to set the minimum password length
  • Runs `gpupdate /force` so SecPol.msc immediately reflects the change
  • Checks for any pending reboot markers and reports status

.Usage
  Must be run in an elevated (Administrator) session.
#>

#region — set the minimum password length
$minLength = 14
Write-Host "Setting Minimum password length to $minLength…" -ForegroundColor Cyan

# net accounts directly writes the local password policy
& net accounts /MINPWLEN:$minLength 2>&1 | ForEach-Object { Write-Host $_ }

if ($LASTEXITCODE -ne 0) {
    Write-Error "❌ Failed to set Minimum password length (exit code $LASTEXITCODE)."
    exit 1
}
Write-Host "✅ Minimum password length set to $minLength." -ForegroundColor Green
#endregion

#region — refresh policy so GUI shows it immediately
Write-Host "Refreshing local Group Policy…" -ForegroundColor Cyan
& gpupdate /force | Out-Null
Write-Host "✅ Group Policy refreshed." -ForegroundColor Green
#endregion

#region — pending reboot detection
function Test-PendingReboot {
    # CBS/Component Based Servicing
    $cbs   = Test-Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending'
    # Windows Update
    $wu    = Test-Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired'
    # Pending file rename operations
    $pfro  = (Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager' `
                -Name PendingFileRenameOperations -ErrorAction SilentlyContinue).PendingFileRenameOperations
    # Computer rename pending
    $cr    = (Get-CimInstance -ClassName Win32_ComputerSystem -ErrorAction SilentlyContinue).RenamePending

    return ($cbs -or $wu -or $pfro -or $cr)
}

if ( Test-PendingReboot ) {
    Write-Warning "⚠️ A reboot is pending. Please restart to complete all changes."
} else {
    Write-Host "ℹ️ No reboot is required for this change." -ForegroundColor Green
}
#endregion
```