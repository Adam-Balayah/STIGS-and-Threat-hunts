```
<#
.SYNOPSIS
    Implements DISA STIG WN10-SO-000250: “User Account Control: Behavior of the elevation prompt for administrators in Admin Approval Mode” must be set to “Prompt for consent on the secure desktop.”


.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000250

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5
#>

# --- Variables ---
$regPath = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System'

# --- Apply the STIG setting ---
Write-Host "Applying STIG setting WN10-SO-000250..." -ForegroundColor Cyan

# 1) Behavior of the elevation prompt for administrators = Prompt for consent on the secure desktop (2)
Set-ItemProperty -Path $regPath -Name 'ConsentPromptBehaviorAdmin' -Value 2 -Type DWord -Force

# 2) Ensure secure desktop is used when prompting
Set-ItemProperty -Path $regPath -Name 'PromptOnSecureDesktop'     -Value 1 -Type DWord -Force

# 3) Refresh group policy so the Local Security Policy snap-in and GUI reflect the change
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
Write-Host "`nVerifying registry settings..." -ForegroundColor Cyan
Test-RegistryPolicy -Path $regPath -Name 'ConsentPromptBehaviorAdmin' -ExpectedValue 2
Test-RegistryPolicy -Path $regPath -Name 'PromptOnSecureDesktop'     -ExpectedValue 1

# --- Reboot Notice ---
Write-Host "`nReboot Required? No." -ForegroundColor Cyan

```