```
<#
.SYNOPSIS
    Configure Windows Data Execution Prevention (DEP) to at least OptOut, per STIG WN10-00-000145.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-00-000145

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

.Usage
    Tested on Windows 10.  
    Requires the BitLocker PowerShell module (ships with Windows 10).
#>

# Ensure running as Administrator
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole(`
        [Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Error "This script must be run as Administrator."
    exit 1
}

# Function to suspend BitLocker on C:
function Suspend-BitLockerIfProtected {
    param(
        [string] $MountPoint = "C:"
    )
    try {
        $vol = Get-BitLockerVolume -MountPoint $MountPoint -ErrorAction Stop
    } catch {
        Write-Verbose "BitLocker module not available or volume not found."
        return
    }
    if ($vol.ProtectionStatus -eq "On") {
        Write-Output "Suspending BitLocker protectors on $MountPoint..."
        Suspend-BitLocker -MountPoint $MountPoint -RebootCount 1
        $script:BitLockerSuspended = $true
    } else {
        Write-Output "BitLocker not protecting $MountPoint; no need to suspend."
    }
}

# Function to resume BitLocker on C:
function Resume-BitLockerIfSuspended {
    param(
        [string] $MountPoint = "C:"
    )
    if ($script:BitLockerSuspended) {
        Write-Output "Re-enabling BitLocker protectors on $MountPoint..."
        Resume-BitLocker -MountPoint $MountPoint
    }
}

# MAIN
Write-Output "== DEP Configuration Script =="
Suspend-BitLockerIfProtected

# Read current DEP setting
$nxLine = bcdedit /enum {current} | Select-String 'nx\s+(\S+)' | ForEach-Object {
    $_.Matches[0].Groups[1].Value
}

if (-not $nxLine) {
    Write-Warning "Could not detect current DEP setting; proceeding to set OptOut."
    $currentSetting = ""
} else {
    $currentSetting = $nxLine.Trim()
    Write-Output "Current DEP setting: $currentSetting"
}

# Only change if less than OptOut
if ($currentSetting -eq 'AlwaysOn' -or $currentSetting -eq 'OptOut') {
    Write-Output "DEP is already set to '$currentSetting'.  No change needed."
} else {
    Write-Output "Setting DEP to OptOut..."
    & bcdedit /set "{current}" nx OptOut 2>&1 | ForEach-Object {
        Write-Output $_
    }
    if ($LASTEXITCODE -eq 0) {
        Write-Output "DEP successfully set to OptOut."
    } else {
        Write-Error "Failed to set DEP.  Please check bcdedit output above."
    }
}

Resume-BitLockerIfSuspended

# Prompt for reboot if setting changed
if ($currentSetting -ne 'AlwaysOn' -and $currentSetting -ne 'OptOut') {
    Write-Warning "You must reboot for the DEP change to take effect."
    $resp = Read-Host "Reboot now? (Y/N)"
    if ($resp -match '^[Yy]') {
        Write-Output "Rebooting..."
        Restart-Computer
    } else {
        Write-Output "Remember to reboot later to apply DEP changes."
    }
}

Write-Output "Done."


```