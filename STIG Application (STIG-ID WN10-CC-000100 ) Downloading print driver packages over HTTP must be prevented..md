```
<#
.SYNOPSIS
  STIG WN10-CC-000100: Prevent downloading of print driver packages over HTTP

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-CC-000100

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# Variables for the policy
$regPath      = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\System\Internet Communication Management\Internet Communication settings'
$regName      = 'TurnOffPrintDriverDownloadOverHttp'
$desiredValue = 1

# 1) Create the registry key if it doesn't already exist
if (-not (Test-Path $regPath)) {
    Write-Host "Creating registry path $regPath…" -ForegroundColor Cyan
    New-Item -Path $regPath -Force | Out-Null
}

# 2) Write the desired DWORD value
Write-Host "Setting `$regName` = $desiredValue at `$regPath`…" -ForegroundColor Cyan
Set-ItemProperty -Path $regPath -Name $regName -Value $desiredValue -Type DWord -Force

# 3) Refresh Group Policy
Write-Host "Refreshing Computer policies (gpupdate)…" -ForegroundColor Cyan
Start-Process -FilePath 'gpupdate.exe' -ArgumentList '/target:computer /force' -Wait -NoNewWindow
$gpExit = $LASTEXITCODE

switch ($gpExit) {
    0 { Write-Host 'GPUpdate completed successfully. No reboot required.' -ForegroundColor Green }
    2 { Write-Host 'GPUpdate completed with errors; a reboot *may* be required.' -ForegroundColor Yellow }
    3 { Write-Host 'GPUpdate requests a reboot for changes to take effect.' -ForegroundColor Yellow }
    default { Write-Host "gpupdate.exe exited with code $gpExit; please review." -ForegroundColor Yellow }
}

# 4) Verification function
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
            Write-Host "PASS: Registry `$Name` found at `$Path` with value `$actual`." -ForegroundColor Green
        }
        else {
            Write-Host "FAIL: Registry `$Name` at `$Path` has value `$actual` (expected `$ExpectedValue`)." -ForegroundColor Red
        }
    }
    catch {
        Write-Host "FAIL: Unable to read `$Path\$Name`." -ForegroundColor Red
    }
}

# Invocation of the verification
Test-RegistryPolicy `
    -Path $regPath `
    -Name $regName `
    -ExpectedValue $desiredValue

```