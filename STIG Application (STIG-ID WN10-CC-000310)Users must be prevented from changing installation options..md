```
<#
.SYNOPSIS
  Applies STIG V-220856 (WN10-CC-000310) fix by disabling user control over installs.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-CC-000310

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

Write-Host "=== Applying STIG: V-220856 (Prevent users from changing installation options) ==="

try {
    # Define the target registry path and value
    $RegPath  = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Installer"
    $RegName  = "EnableUserControl"
    $RegValue = 0

    # Create the key if it does not exist
    if (-not (Test-Path $RegPath)) {
        Write-Host "Registry path does not exist. Creating path: $RegPath"
        New-Item -Path $RegPath -Force | Out-Null
    }

    # Set the registry value
    Write-Host "Setting $RegName to $RegValue under $RegPath..."
    New-ItemProperty -Path $RegPath -Name $RegName -PropertyType DWORD -Value $RegValue -Force | Out-Null

    Write-Host "Successfully set 'Allow user control over installs' to Disabled (EnableUserControl=0)."
} 
catch {
    Write-Error "Failed to set the registry value. Error details: $_"
}

```