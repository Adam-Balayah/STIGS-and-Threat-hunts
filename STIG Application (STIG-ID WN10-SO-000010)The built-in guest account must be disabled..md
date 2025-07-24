```
<#
.SYNOPSIS
  Disables the built-in Guest account to comply with STIG WN10-SO-000010.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000010

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# Requires at least PowerShell 5.1 for the *-LocalUser cmdlets.

# Check if the current user is running the script with administrator privileges
# If not, display a warning and exit.
$currentUser = [Security.Principal.WindowsIdentity]::GetCurrent()
$principal = New-Object Security.Principal.WindowsPrincipal($currentUser)
if (-not $principal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Host "This script must be run with Administrator privileges. Please run an elevated PowerShell session and try again."
    Exit 1
}

try {
    # Try to get the local 'Guest' account
    $guestAccount = Get-LocalUser -Name "Guest" -ErrorAction Stop

    if ($guestAccount.Enabled -eq $true) {
        # If the Guest account is enabled, disable it
        Disable-LocalUser -Name "Guest"
        Write-Host "Guest account was enabled and has now been disabled."
    }
    else {
        Write-Host "Guest account is already disabled."
    }
}
catch {
    Write-Host "Failed to retrieve or modify the Guest account. Error details:"
    Write-Host $_.Exception.Message
    Exit 1
}

Write-Host "Script completed."

```