```
<#
.SYNOPSIS
  Ensures the SMB client always signs SMB packets (WN10-SO-000100).

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000100

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5
#>

# Parameters for this STIG setting
$RegPath       = 'HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters'
$ValueName     = 'RequireSecuritySignature'
$DesiredValue  = 1

# 1. Apply the registry setting
if (-not (Test-Path $RegPath)) {
    New-Item -Path $RegPath -Force | Out-Null
}
Set-ItemProperty -Path $RegPath -Name $ValueName -Value $DesiredValue -Type DWord
Write-Host "Applied: `$ValueName = $DesiredValue at $RegPath" -ForegroundColor Cyan

# 2. Refresh Group Policy so the Local Security Policy GUI shows the new setting
Write-Host "Refreshing Group Policy..." -NoNewline
gpupdate /force | Out-Null
Write-Host " Done." -ForegroundColor Cyan

# 3. Verification function for any registry-based policy
function Test-RegistryPolicy {
    param(
        [Parameter(Mandatory)][string] $Path,
        [Parameter(Mandatory)][string] $Name,
        [Parameter(Mandatory)][ValidateNotNullOrEmpty()][Alias('Expected')] $ExpectedValue
    )
    try {
        $actual = Get-ItemProperty -Path $Path -Name $Name -ErrorAction Stop |
                  Select-Object -ExpandProperty $Name
        if ($actual -eq $ExpectedValue) {
            Write-Host "PASS: Registry `$Name` at `$Path` = `$actual` (as expected)." -ForegroundColor Green
        }
        else {
            Write-Host "FAIL: Registry `$Name` at `$Path` = `$actual` (expected `$ExpectedValue`)." -ForegroundColor Red
        }
    }
    catch {
        Write-Host "FAIL: Cannot read `$Path`\`$Name`." -ForegroundColor Red
    }
}

# 4. Invocation of the verification for this specific policy
Test-RegistryPolicy `
    -Path $RegPath `
    -Name $ValueName `
    -ExpectedValue $DesiredValue

# 5. Check for pending reboot
function Test-PendingReboot {
    $checks = @(
        'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending',
        'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired',
        'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\PendingFileRenameOperations'
    )
    foreach ($p in $checks) {
        if (Test-Path $p) { return $true }
    }
    return $false
}

if (Test-PendingReboot) {
    Write-Host "REBOOT REQUIRED: A system restart is needed for this setting to fully take effect." -ForegroundColor Yellow
}
else {
    Write-Host "No reboot required." -ForegroundColor Green
}

```