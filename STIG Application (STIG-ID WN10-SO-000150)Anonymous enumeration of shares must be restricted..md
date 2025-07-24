```
<#
.SYNOPSIS
  Implements STIG WN10-SO-000150: Restrict anonymous enumeration of SAM accounts and shares.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000150

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# Variables
$regPath      = 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa'
$regName      = 'restrictanonymous'
$desiredValue = 1

# 1) Apply the setting
if (-not (Test-Path $regPath)) {
    New-Item -Path $regPath -Force | Out-Null
}
Set-ItemProperty -Path $regPath -Name $regName -Value $desiredValue -Type DWord
Write-Host "Applied: [$regName] = $desiredValue at $regPath" -ForegroundColor Green

# 2) Refresh policy
Write-Host "Refreshing Group Policy…" -ForegroundColor Cyan
gpupdate /force | Out-Null
Write-Host "Policy refresh complete." -ForegroundColor Cyan

# 3) Registry-verification function
function Test-RegistryPolicy {
    param(
        [Parameter(Mandatory)][string]$Path,
        [Parameter(Mandatory)][string]$Name,
        [Parameter(Mandatory)][ValidateNotNullOrEmpty()] $ExpectedValue
    )
    try {
        $actual = Get-ItemProperty -Path $Path -Name $Name -ErrorAction Stop |
                  Select-Object -ExpandProperty $Name
        if ($actual -eq $ExpectedValue) {
            Write-Host "PASS: Registry '$Name' at '$Path' = $actual" -ForegroundColor Green
        }
        else {
            Write-Host "FAIL: Registry '$Name' at '$Path' = $actual (expected $ExpectedValue)" -ForegroundColor Red
        }
    }
    catch {
        Write-Host "FAIL: Cannot read '$Path\$Name'" -ForegroundColor Red
    }
}

# Invoke registry check
Test-RegistryPolicy `
    -Path $regPath `
    -Name $regName `
    -ExpectedValue $desiredValue

# 4) Reboot-pending detection
function Test-PendingReboot {
    $pending = $false
    # Windows Update
    try {
        $wu = Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update' `
                               -Name RebootRequired -ErrorAction SilentlyContinue
        if ($wu.RebootRequired) { $pending = $true }
    } catch {}
    # Component Based Servicing
    if (Test-Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending') {
        $pending = $true
    }
    # Pending file rename operations
    try {
        $session = Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager' `
                                   -Name 'PendingFileRenameOperations' -ErrorAction SilentlyContinue
        if ($session.PendingFileRenameOperations) { $pending = $true }
    } catch {}
    return $pending
}

if (Test-PendingReboot) {
    Write-Host "NOTICE: A system reboot is required for this change to take effect." -ForegroundColor Yellow
} else {
    Write-Host "No reboot is currently pending." -ForegroundColor Green
}
```