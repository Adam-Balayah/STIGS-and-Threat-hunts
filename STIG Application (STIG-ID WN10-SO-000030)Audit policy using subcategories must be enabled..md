```
<#
.SYNOPSIS
This script ensures that Windows 10 is configured to use audit policy subcategories, enabling more granular and effective audit logging.


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

# Define registry path  
$RegPath = "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa"  
# Ensure the registry path exists  
If (!(Test-Path $RegPath)) {  
New-Item -Path $RegPath -Force  
}  
# Set SCENoApplyLegacyAuditPolicy to 1 (Enable policy override)  
Set-ItemProperty -Path $RegPath -Name "SCENoApplyLegacyAuditPolicy" -Value 1 -Type DWord  
# Force Group Policy update to apply changes  
gpupdate /force  
# Verify the configuration  
Get-ItemProperty -Path $RegPath -Name "SCENoApplyLegacyAuditPolicy"
```