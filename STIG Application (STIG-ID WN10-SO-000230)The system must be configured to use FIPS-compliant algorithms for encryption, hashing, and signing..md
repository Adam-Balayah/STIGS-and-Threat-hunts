```
<#
.SYNOPSIS
  Enable FIPS-compliant algorithms for encryption, hashing, and signing.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000230

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

Write-Host "1) Configuring FIPS-compliant algorithms policy…" -ForegroundColor Cyan

#----------------------------------------
# 1) Ensure the registry path exists and set Enabled = 1
#----------------------------------------
$RegPath = 'HKLM:\System\CurrentControlSet\Control\Lsa\FipsAlgorithmPolicy'
if (-not (Test-Path $RegPath)) {
    Write-Host "   Creating registry key: $RegPath" -ForegroundColor Cyan
    New-Item -Path $RegPath -Force | Out-Null
}

Write-Host "   Setting 'Enabled' = 1 (FIPS enabled)" -ForegroundColor Cyan
Set-ItemProperty -Path $RegPath -Name 'Enabled' -Value 1 -Type DWord

#----------------------------------------
# 2) (Optional) Refresh Group Policy so GUI-scanners pick it up
#----------------------------------------
Write-Host "2) Refreshing Group Policy…" -ForegroundColor Cyan
gpupdate.exe /force | Out-Null

#----------------------------------------
# 3) Verification function
#----------------------------------------
function Test-RegistryPolicy {
    param(
        [Parameter(Mandatory)][string]$Path,
        [Parameter(Mandatory)][string]$Name,
        [Parameter(Mandatory)][ValidateNotNullOrEmpty()] $ExpectedValue
    )
    try {
        $actual = Get-ItemProperty -Path $Path -Name $Name -ErrorAction Stop | Select-Object -ExpandProperty $Name
        if ($actual -eq $ExpectedValue) {
            Write-Host "PASS: Registry '$Name' found at '$Path' with value '$actual'." -ForegroundColor Green
        }
        else {
            Write-Host "FAIL: Registry '$Name' at '$Path' has value '$actual' (expected '$ExpectedValue')." -ForegroundColor Red
        }
    }
    catch {
        Write-Host "FAIL: Unable to read '$Path\$Name'." -ForegroundColor Red
    }
}

#----------------------------------------
# 4) Run the verification
#----------------------------------------
Write-Host "3) Verifying registry setting…" -ForegroundColor Cyan
Test-RegistryPolicy `
    -Path $RegPath `
    -Name 'Enabled' `
    -ExpectedValue 1

#----------------------------------------
# 5) Reboot guidance
#----------------------------------------
Write-Host "`n4) Reboot required? No. The FIPS policy is enforced immediately." -ForegroundColor Yellow

```