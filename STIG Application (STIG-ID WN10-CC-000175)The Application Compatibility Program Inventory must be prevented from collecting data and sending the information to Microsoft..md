```
<#
.SYNOPSIS
    This script ensures that the Application Compatibility Program Inventory is disabled to prevent data collection and transmission to Microsoft on Windows 10 systems.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-CC-000175

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5
#>


# Define registry path  
$RegPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\AppCompat"  
# Ensure the registry path exists  
If (!(Test-Path $RegPath)) {  
New-Item -Path $RegPath -Force  
}  
# Set DisableInventory to 1 (Turn off Inventory Collector)  
Set-ItemProperty -Path $RegPath -Name "DisableInventory" -Value 1 -Type DWord  
# Force Group Policy update to apply changes  
gpupdate /force  
# Verify the configuration  
Get-ItemProperty -Path $RegPath -Name "DisableInventory"
```