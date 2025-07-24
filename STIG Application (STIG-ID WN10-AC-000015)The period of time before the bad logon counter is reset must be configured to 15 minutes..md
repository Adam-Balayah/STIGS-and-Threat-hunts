```
<#
.SYNOPSIS
  Configure “Reset account lockout counter after” to 15 minutes.
  
.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-AC-000015

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

Write-Host "1) Reading current lockout threshold…" -ForegroundColor Cyan
$output = net accounts 2>&1
$thrLine = $output | Select-String 'Lockout threshold'
if (-not $thrLine) {
    Write-Error "Unable to read current lockout threshold."
    exit 1
}
$threshold = [int]($thrLine -replace '.*?:\s*','').Trim()
Write-Host "   Current threshold = $threshold" -ForegroundColor Cyan

if ($threshold -eq 0) {
    Write-Error "Lockout threshold is 0 (lockout disabled). Set AC-000014 first."
    exit 1
}

Write-Host "2) Setting duration = 15 min and reset window = 15 min…" -ForegroundColor Cyan
net accounts `
    /lockoutthreshold:$threshold `
    /lockoutduration:15 `
    /lockoutwindow:15

Write-Host "3) Verifying settings…" -ForegroundColor Cyan
$output = net accounts 2>&1

# Parse duration
$dur = ($output |
    Select-String 'Lockout duration \(minutes\):\s*(\d+)' |
    ForEach-Object { $_.Matches[0].Groups[1].Value }) -as [int]

# Parse window
$win = ($output |
    Select-String 'Lockout observation window \(minutes\):\s*(\d+)' |
    ForEach-Object { $_.Matches[0].Groups[1].Value }) -as [int]

if ($dur -eq 15 -and $win -eq 15) {
    Write-Host "PASS: Duration and reset window are both 15 minutes." -ForegroundColor Green
}
else {
    Write-Host "FAIL: Duration is $dur min (expected 15) and window is $win min (expected 15)." -ForegroundColor Red
}

Write-Host "`n4) Reboot required? No. Changes apply immediately." -ForegroundColor Yellow
```