```
<#
.SYNOPSIS
    Configure Windows SMB Server to always perform SMB packet signing (STIG ID WN10-SO-000120).


.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000120

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# Variables for the policy
$registryPath   = 'HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters'
$propertyName   = 'RequireSecuritySignature'
$desiredValue   = 1

Write-Host "Applying STIG WN10-SO-000120: Enforce SMB packet signing on the server..." -ForegroundColor Cyan

# 1) Apply the registry setting
try {
    Set-ItemProperty -Path $registryPath -Name $propertyName -Value $desiredValue -Type DWord -ErrorAction Stop
    Write-Host "✔ SUCCESS: Set `$propertyName` = `$desiredValue` at `$registryPath`." -ForegroundColor Green
}
catch {
    Write-Host "✖ ERROR: Failed to set `$propertyName` at `$registryPath`: $_" -ForegroundColor Red
    exit 1
}

# 2) Force a Group Policy update so the GUI snaps to the new value
Write-Host "Refreshing Group Policy..."
& gpupdate /target:computer /force | Out-Null

# 3) Restart the SMB server service so the change takes effect immediately
Write-Host "Restarting LanmanServer service..."
try {
    Restart-Service -Name LanmanServer -Force -ErrorAction Stop
    Write-Host "✔ LanmanServer service restarted successfully. No reboot required." -ForegroundColor Green
}
catch {
    Write-Host "⚠ WARNING: Could not restart LanmanServer service. A reboot may be required to apply the change." -ForegroundColor Yellow
}

# 4) Registry‐verification function (PASS/FAIL)
function Test-RegistryPolicy {
    param(
        [Parameter(Mandatory)][string] $Path,
        [Parameter(Mandatory)][string] $Name,
        [Parameter(Mandatory)][ValidateNotNullOrEmpty()] $ExpectedValue
    )
    try {
        $actual = Get-ItemProperty -Path $Path -Name $Name -ErrorAction Stop | Select-Object -ExpandProperty $Name
        if ($actual -eq $ExpectedValue) {
            Write-Host "PASS: Registry '$Name' at '$Path' = '$actual'." -ForegroundColor Green
        }
        else {
            Write-Host "FAIL: Registry '$Name' at '$Path' = '$actual' (expected '$ExpectedValue')." -ForegroundColor Red
        }
    }
    catch {
        Write-Host "FAIL: Unable to read registry '$Path\$Name'." -ForegroundColor Red
    }
}

# Invoke the test
Test-RegistryPolicy `
    -Path $registryPath `
    -Name $propertyName `
    -ExpectedValue $desiredValue

```