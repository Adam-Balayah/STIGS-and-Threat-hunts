```
<#
.SYNOPSIS
  Implements DISA STIG WN10-AC-000010 by configuring the Local Security Policy
  (Account Lockout Policy → Account lockout threshold) to 3 invalid attempts.
  Uses the built-in net accounts command so that the GUI (secpol.msc) reflects
  the change immediately.
  
.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-AC-000010

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# 1) Read current lockout threshold
$netOutput = net accounts 2>&1
$thresholdLine = $netOutput | Where-Object { $_ -match 'Lockout threshold' }
if (-not $thresholdLine) {
    Write-Error "Unable to determine current lockout threshold."
    exit 1
}
# Parse the numeric value from the line
$currentThreshold = [int]($thresholdLine -replace '.*:\s*','')

Write-Host "Current lockout threshold: $currentThreshold"

# 2) Determine if change is needed (must be between 1 and 3 inclusive; 0 is unacceptable)
if ($currentThreshold -ge 1 -and $currentThreshold -le 3) {
    Write-Host "✔ The current setting ($currentThreshold) meets the STIG requirement (1–3)."
}
else {
    # Set threshold to 3 invalid logon attempts
    Write-Host "✖ Threshold of $currentThreshold does not comply. Setting to 3..."
    $setResult = net accounts /lockoutthreshold:3 2>&1
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✔ Successfully set lockout threshold to 3."
    }
    else {
        Write-Error "Failed to set lockout threshold. Details:`n$setResult"
        exit 1
    }
}

# 3) Confirm the GUI (Local Security Policy) is updated
#    The net accounts command writes the policy to the Local Security Policy store,
#    which is immediately reflected in secpol.msc → Account Policies → Account Lockout Policy.

# 4) Reboot requirement
Write-Host "`nℹ️  No reboot is required for this setting to take effect."

```