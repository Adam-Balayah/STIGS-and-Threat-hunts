```
<#
.SYNOPSIS
    Implements DISA STIG WN10-SO-000070: Set Interactive logon: Machine inactivity limit to 900 seconds.


.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000070

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# Variables for the policy
$RegPath   = 'HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System'
$ValueName = 'InactivityTimeoutSecs'
$Desired   = 900

# 1. Ensure the registry key exists
If (-Not (Test-Path $RegPath)) {
    New-Item -Path $RegPath -Force | Out-Null
}

# 2. Write the desired value
Write-Host "Setting machine inactivity limit to $Desired seconds..." -ForegroundColor Cyan
Set-ItemProperty -Path $RegPath -Name $ValueName -Value $Desired -Type DWord -Force

# 3. Refresh Group Policy so the Local Security Policy GUI updates
Write-Host "Refreshing Group Policy..." -ForegroundColor Cyan
& gpupdate.exe /target:computer /force | Out-Null

# 4. Define the registry verification function
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

# 5. Invoke the verification
Test-RegistryPolicy `
    -Path $RegPath `
    -Name $ValueName `
    -ExpectedValue $Desired

# 6. Note on reboot requirement
Write-Host ""
Write-Host "NOTE: No reboot is required. The setting takes effect after the Group Policy refresh." -ForegroundColor Yellow
```