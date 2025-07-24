```
<#
.SYNOPSIS
  Configure BitLocker pre-boot PIN for OS drive to comply with STIG WN10-00-000031.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-00-000031

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# Ensure script is run as administrator
$currentUser = [Security.Principal.WindowsIdentity]::GetCurrent()
$principal = New-Object Security.Principal.WindowsPrincipal($currentUser)
if (-not $principal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Host "ERROR: This script must be run with Administrator privileges."
    Exit 1
}

# Registry path for BitLocker policy
$registryPath = "HKLM:\SOFTWARE\Policies\Microsoft\FVE"

# Set values for standard TPM + PIN (use '2' for both if using BitLocker Network Unlock + PIN)
$useAdvancedStartupValue = 1
$useTPMPINValue          = 1
$useTPMKeyPINValue       = 1

# Create policy key if it doesn't exist
if (-not (Test-Path $registryPath)) {
    New-Item -Path $registryPath | Out-Null
}

# Set required registry values
New-ItemProperty -Path $registryPath -Name "UseAdvancedStartup" -Value $useAdvancedStartupValue -PropertyType DWord -Force | Out-Null
Write-Host "UseAdvancedStartup set to $useAdvancedStartupValue"

New-ItemProperty -Path $registryPath -Name "UseTPMPIN" -Value $useTPMPINValue -PropertyType DWord -Force | Out-Null
Write-Host "UseTPMPIN set to $useTPMPINValue"

New-ItemProperty -Path $registryPath -Name "UseTPMKeyPIN" -Value $useTPMKeyPINValue -PropertyType DWord -Force | Out-Null
Write-Host "UseTPMKeyPIN set to $useTPMKeyPINValue"

Write-Host "`nBitLocker PIN policy settings updated successfully."

```