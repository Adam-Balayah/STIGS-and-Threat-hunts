```
<#
.SYNOPSIS
    Ensures the Local Security Policy "Minimum password age" is at least 1 day.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-AC-000030

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

#region Configuration
$DesiredMinAge = 1
#endregion

#region Helper Functions

function Get-MinPasswordAge {
    <#
    .SYNOPSIS
        Gets the current Minimum password age in days.
    .OUTPUTS
        [int] Current minimum password age
    .NOTES
        Tries `net accounts` first; if that fails, exports via secedit.
    #>
    Write-Verbose "Trying to read via net accounts..."
    $na = net accounts 2>&1
    foreach ($line in $na) {
        if ($line -match '(?i)Minimum\s+password\s+age\b.*:\s*(\d+)') {
            return [int]$Matches[1]
        }
    }

    Write-Warning "Could not parse `net accounts` output. Falling back to secedit export..."
    $cfg = Join-Path $env:TEMP 'secpol.cfg'
    secedit /export /cfg $cfg | Out-Null

    try {
        $sec = Get-Content $cfg
        foreach ($line in $sec) {
            if ($line -match '^\s*MinimumPasswordAge\s*=\s*(\d+)\s*$') {
                return [int]$Matches[1]
            }
        }
        throw "No MinimumPasswordAge line found in exported policy."
    }
    finally {
        Remove-Item $cfg -ErrorAction SilentlyContinue
    }
}

function Set-MinPasswordAge {
    param (
        [Parameter(Mandatory)][int]$Days
    )
    Write-Verbose "Setting minimum password age to $Days..."
    net accounts /minpwage:$Days | Out-Null
}

#endregion

try {
    Write-Host "`n=== Checking Minimum password age ===" -ForegroundColor Cyan
    $currentAge = Get-MinPasswordAge
    Write-Host "Current Minimum password age: $currentAge day(s)."

    if ($currentAge -lt $DesiredMinAge) {
        Write-Host "Desired is $DesiredMinAge. Updating..." -ForegroundColor Yellow
        Set-MinPasswordAge -Days $DesiredMinAge

        # re-verify
        $newAge = Get-MinPasswordAge
        if ($newAge -eq $DesiredMinAge) {
            Write-Host "✔ Successfully set Minimum password age to $newAge day(s)." -ForegroundColor Green
        }
        else {
            Write-Error "✖ Failed to set Minimum password age. Current is $newAge day(s)."
            exit 1
        }
    }
    else {
        Write-Host "✔ Already compliant (≥ $DesiredMinAge day(s)). No change needed." -ForegroundColor Green
    }

    Write-Host "`nℹ Password policy updates apply immediately; no reboot required." -ForegroundColor Cyan
    exit 0
}
catch {
    Write-Error "An error occurred: $_"
    exit 1
}

```