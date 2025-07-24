```
<#
.SYNOPSIS
  Implements WN10-SO-000255: UAC automatically denies elevation for standard users.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000255

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5
#>

# Variables
$registryPath = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System'
$valueName    = 'ConsentPromptBehaviorUser'
$desiredValue = 0

# 1. Apply the setting
Write-Host "Applying STIG setting: $valueName = $desiredValue at $registryPath..."
try {
    # Ensure key exists
    if (-not (Test-Path $registryPath)) {
        New-Item -Path $registryPath -Force | Out-Null
    }
    # Set the value
    Set-ItemProperty -Path $registryPath -Name $valueName -Value $desiredValue -Type DWord -ErrorAction Stop
    Write-Host "✔ Registry updated successfully." -ForegroundColor Green
}
catch {
    Write-Host "✖ ERROR: Failed to set registry value: $_" -ForegroundColor Red
    exit 1
}

# 2. Refresh group policy to update GUI/policy store
Write-Host "`nForcing Group Policy update..."
try {
    gpupdate.exe /target:computer /force /wait:0 | Out-Null
    Write-Host "✔ gpupdate completed." -ForegroundColor Green
}
catch {
    Write-Host "⚠ WARNING: gpupdate failed: $_" -ForegroundColor Yellow
}

# 3. Inform about reboot requirements
Write-Host "`nNOTE: Changes to UAC policies typically require a reboot (or logoff) to take full effect." -ForegroundColor Yellow

# 4. Registry-verification snippet
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

# 5. Invoke the test
Write-Host "`nVerifying registry setting..."
Test-RegistryPolicy `
    -Path $registryPath `
    -Name $valueName `
    -ExpectedValue $desiredValue

```