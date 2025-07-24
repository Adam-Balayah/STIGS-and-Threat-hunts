```
<#
.SYNOPSIS
    DISA STIG WN10-SO-000080: Configure the Windows dialog box title for the legal banner.

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
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5
#>

# 1. Variables
$RegPath      = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System'
$ValueName    = 'LegalNoticeCaption'
$DesiredValue = 'DoD Notice and Consent Banner'

# 2. Ensure the registry key path exists
if (-not (Test-Path $RegPath)) {
    Write-Host "Creating registry key path: $RegPath" -ForegroundColor Cyan
    New-Item -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies' `
             -Name 'System' -Force | Out-Null
}

# 3. Set the registry value for the logon banner title
Write-Host "Setting '$ValueName' to '$DesiredValue' under $RegPath" -ForegroundColor Cyan
Set-ItemProperty -Path $RegPath -Name $ValueName -Value $DesiredValue -Type String -Force

# 4. Refresh Group Policy so the Local Security Policy GUI reflects the change
Write-Host "Refreshing Group Policy..." -ForegroundColor Cyan
gpupdate /force | Out-Null
Write-Host "Group Policy refreshed." -ForegroundColor Green

# 5. Verification function
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
            Write-Host "PASS: Registry '$Name' at '$Path' = '$actual'." -ForegroundColor Green
        }
        else {
            Write-Host "FAIL: Registry '$Name' at '$Path' = '$actual' (expected '$ExpectedValue')." -ForegroundColor Red
        }
    }
    catch {
        Write-Host "FAIL: Unable to read '$Path\$Name'." -ForegroundColor Red
    }
}

# 6. Invoke verification
Test-RegistryPolicy `
    -Path $RegPath `
    -Name $ValueName `
    -ExpectedValue $DesiredValue

# 7. Reboot requirement notice
Write-Host "INFO: No reboot is required for this setting to take effect." -ForegroundColor Yellow

```