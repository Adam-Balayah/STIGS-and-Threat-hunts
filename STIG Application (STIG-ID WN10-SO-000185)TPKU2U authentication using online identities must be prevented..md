```<#
.SYNOPSIS
    DISA STIG WN10-SO-000185: Prevent PKU2U authentication using online identities.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000185

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# 1. Variables
$RegPath  = 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\pku2u'
$ValueName = 'AllowOnlineID'
$DesiredValue = 0

# 2. Ensure the registry key path exists
if (-not (Test-Path $RegPath)) {
    Write-Host "Creating registry key path: $RegPath" -ForegroundColor Cyan
    New-Item -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa' -Name 'pku2u' -Force | Out-Null
}

# 3. Set the registry value to disable PKU2U online identities
Write-Host "Setting $ValueName to $DesiredValue under $RegPath" -ForegroundColor Cyan
Set-ItemProperty -Path $RegPath -Name $ValueName -Value $DesiredValue -Type DWord -Force

# 4. Refresh Group Policy so the GUI/Local Security Policy snap-in reflects the change
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

# 6. Invoke verification
Test-RegistryPolicy `
    -Path $RegPath `
    -Name $ValueName `
    -ExpectedValue $DesiredValue

# 7. Reboot requirement notice
Write-Host "INFO: No reboot is required for this setting to take effect." -ForegroundColor Yellow

```