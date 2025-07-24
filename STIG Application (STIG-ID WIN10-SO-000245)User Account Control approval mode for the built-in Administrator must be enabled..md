```
<#
.SYNOPSIS
    Implements DISA STIG WN10-SO-000245: “User Account Control: Admin Approval Mode for the Built-in Administrator account” must be enabled.
    
.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WIN10-SO-000245

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

.DESCRIPTION
    • Sets the required registry value under HKLM:\…\Policies\System  
    • Forces a Group Policy update so the GUI reflects the change immediately  
    • Verifies the setting via a PASS/FAIL registry check  
    • Reports whether a reboot is needed  


#>

# --- Variables ---
$regPath = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System'
$valueName = 'FilterAdministratorToken'
$desiredValue = 1

# --- Apply the STIG setting ---
Write-Host "Applying STIG setting WN10-SO-000245..." -ForegroundColor Cyan

Set-ItemProperty -Path $regPath -Name $valueName -Value $desiredValue -Type DWord -Force

Write-Host "Refreshing Group Policy..." -NoNewline
gpupdate /force | Out-Null
Write-Host " Done." -ForegroundColor Green

# --- Registry Verification Function ---
function Test-RegistryPolicy {
    param(
        [Parameter(Mandatory)][string] $Path,
        [Parameter(Mandatory)][string] $Name,
        [Parameter(Mandatory)][ValidateNotNullOrEmpty()] $ExpectedValue
    )
    try {
        $actual = Get-ItemProperty -Path $Path -Name $Name -ErrorAction Stop |
                  Select-Object -ExpandProperty $Name
        if ($actual -eq $ExpectedValue) {
            Write-Host "PASS: '$Name' at '$Path' = $actual" -ForegroundColor Green
        }
        else {
            Write-Host "FAIL: '$Name' at '$Path' = $actual (expected $ExpectedValue)" -ForegroundColor Red
        }
    }
    catch {
        Write-Host "FAIL: Unable to read '$Path\$Name'." -ForegroundColor Red
    }
}

# --- Run Verification ---
Write-Host "`nVerifying registry setting for built-in Administrator UAC..." -ForegroundColor Cyan
Test-RegistryPolicy -Path $regPath -Name $valueName -ExpectedValue $desiredValue

# --- Reboot Notice ---
Write-Host "`nReboot Required? No." -ForegroundColor Cyan
```