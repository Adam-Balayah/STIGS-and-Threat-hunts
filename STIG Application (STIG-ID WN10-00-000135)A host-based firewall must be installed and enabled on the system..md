```
<#
.SYNOPSIS
    Ensure host-based firewall is installed and enabled on Windows 10.
.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-00-000135 

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

.DESCRIPTION
    Implements DISA STIG rule WN10-00-000135 (CCI-000366):
    "A host-based firewall must be installed and enabled on the system."

.Usage
    - Must be run in an elevated (Administrator) PowerShell session.
    - Tested on Windows 10 v1809 and later.
    - References:
        STIG-ID:        WN10-00-000135
        Rule-ID:        SV-220724r569187_rule
        CCI:            CCI-000366
#>

#-----------------------
# Helper: Write a status message
function Write-Status {
    param([string]$Message, [string]$Level = 'INFO')
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    Write-Host "[$timestamp] [$Level] $Message"
}

#-----------------------
# 1. Ensure the Windows Defender Firewall service (MpsSvc) exists, set to Automatic, and running
$svcName = 'MpsSvc'
try {
    Write-Status "Checking for service '$svcName'..."
    $svc = Get-Service -Name $svcName -ErrorAction Stop

    if ($svc.StartType -ne 'Automatic') {
        Write-Status "Setting startup type of '$svcName' to Automatic..."
        Set-Service -Name $svcName -StartupType Automatic
    } else {
        Write-Status "Startup type of '$svcName' is already Automatic."
    }

    if ($svc.Status -ne 'Running') {
        Write-Status "Starting service '$svcName'..."
        Start-Service -Name $svcName
    } else {
        Write-Status "Service '$svcName' is already running."
    }
}
catch {
    Write-Status "ERROR: Windows Defender Firewall service ('$svcName') not found or cannot be controlled. $_" 'ERROR'
    Exit 1
}

#-----------------------
# 2. Import NetSecurity module (provides Set-NetFirewallProfile)
if (-not (Get-Module -Name NetSecurity)) {
    try {
        Write-Status "Importing NetSecurity module..."
        Import-Module NetSecurity -ErrorAction Stop
    }
    catch {
        Write-Status "ERROR: Could not import NetSecurity module. $_" 'ERROR'
        Exit 1
    }
}

#-----------------------
# 3. Enable Firewall on all profiles
$profiles = @('Domain','Private','Public')
foreach ($p in $profiles) {
    try {
        $current = (Get-NetFirewallProfile -Profile $p).Enabled
        if ($current -ne 'True') {
            Write-Status "Enabling Windows Firewall for the $p profile..."
            Set-NetFirewallProfile -Profile $p -Enabled True -ErrorAction Stop
        }
        else {
            Write-Status "Windows Firewall is already enabled on the $p profile."
        }
    }
    catch {
        Write-Status "ERROR: Failed to enable firewall on $p profile. $_" 'ERROR'
    }
}

#-----------------------
# 4. Confirm final state
Write-Status "Final firewall profile states:"
Get-NetFirewallProfile | Select-Object Name, @{n='Enabled';e={if($_.Enabled){'True'}else{'False'}}} |
    Format-Table -AutoSize

Write-Status "STIG WN10-00-000135 compliance remediation complete."

```