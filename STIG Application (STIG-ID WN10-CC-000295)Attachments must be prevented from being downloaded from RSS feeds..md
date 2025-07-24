```
<#
.SYNOPSIS
Prevents attachments from being downloaded from RSS feeds by setting the registry value.
  
.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-CC-000295

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

# Registry path for RSS Feed enclosure download policy
$registryPath = "HKLM:\SOFTWARE\Policies\Microsoft\Internet Explorer\Feeds"
$valueName = "DisableEnclosureDownload"
$valueData = 1

# Create the registry key if it does not exist
if (-not (Test-Path $registryPath)) {
    New-Item -Path $registryPath -Force | Out-Null
}

# Create or update the registry value
New-ItemProperty -Path $registryPath -Name $valueName -Value $valueData -PropertyType DWord -Force | Out-Null
Write-Host "$($valueName) set to $($valueData) under $registryPath"

Write-Host "`nRSS Feed enclosure download prevention configured successfully (STIG WN10-CC-000295)."

```