```
 <#
.SYNOPSIS
  Implements WN10-AC-000020: Enforce password history of 24 remembered passwords.
  
.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-AC-000020

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

#----- VARIABLES --------------------------------------------------------------
$valueName    = 'PasswordHistorySize'
$desiredValue = 24

#----- 1. APPLY THE SETTING --------------------------------------------------
Write-Host "Applying STIG WN10-AC-000020: setting password history to $desiredValue..."
try {
    net accounts /uniquepw:$desiredValue | Out-Null
    Write-Host "✔ Password history configured to $desiredValue." -ForegroundColor Green
}
catch {
    Write-Host "✖ ERROR: Failed to set password history: $_" -ForegroundColor Red
    exit 1
}

#----- 2. REFRESH POLICY -----------------------------------------------------
Write-Host "`nForcing local policy update (so GUI and scanners see it immediately)..."
try {
    gpupdate.exe /force /wait:0 | Out-Null
    Write-Host "✔ Policy refresh complete." -ForegroundColor Green
}
catch {
    Write-Host "⚠ WARNING: gpupdate failed: $_" -ForegroundColor Yellow
}

#----- 3. REBOOT NOTE --------------------------------------------------------
Write-Host "`nNOTE: Password policy changes take effect immediately; no reboot required." -ForegroundColor Yellow

#----- 4. VERIFICATION SNIPPET -----------------------------------------------
function Test-LocalSecurityPolicy {
    param(
        [Parameter(Mandatory)][string]$PolicyName,
        [Parameter(Mandatory)][int]   $ExpectedValue
    )

    $exportPath = "$env:windir\Temp\SecurityPolicy.cfg"
    try {
        # Export the current local policy
        secedit.exe /export /cfg $exportPath /Areas SECURITYPOLICY | Out-Null

        # Parse out the policy line
        $line = Get-Content $exportPath |
                Where-Object { $_ -match "^\s*$PolicyName\s*=" }

        if ($line -match "$PolicyName\s*=\s*(\d+)") {
            $actual = [int]$matches[1]
            if ($actual -eq $ExpectedValue) {
                Write-Host "PASS: $PolicyName is set to $actual." -ForegroundColor Green
            }
            else {
                Write-Host "FAIL: $PolicyName is $actual (expected $ExpectedValue)." -ForegroundColor Red
            }
        }
        else {
            Write-Host "FAIL: $PolicyName not found in exported policy." -ForegroundColor Red
        }
    }
    catch {
        Write-Host "FAIL: Unable to export/read local policy: $_" -ForegroundColor Red
    }
    finally {
        # Clean up temp file
        if (Test-Path $exportPath) { Remove-Item $exportPath -Force }
    }
}

#----- 5. RUN THE TEST --------------------------------------------------------
Write-Host "`nVerifying 'Enforce password history' setting..."
Test-LocalSecurityPolicy -PolicyName $valueName -ExpectedValue $desiredValue

```