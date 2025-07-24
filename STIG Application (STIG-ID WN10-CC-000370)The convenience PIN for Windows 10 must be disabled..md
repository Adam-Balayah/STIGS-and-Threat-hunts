```
<#
.SYNOPSIS
    This script ensures that the Windows 10 convenience PIN sign-in is disabled to enhance domain authentication security.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-CC-000370

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# Define registry path  
$RegPath = "HKLM:\Software\Policies\Microsoft\Windows\System"  
# Ensure the registry path exists  
If (!(Test-Path $RegPath)) {  
New-Item -Path $RegPath -Force  
}  
# Set AllowDomainPINLogon to 0 (Disable convenience PIN sign-in)  
Set-ItemProperty -Path $RegPath -Name "AllowDomainPINLogon" -Value 0 -Type DWord  
# Force Group Policy update to apply changes  
gpupdate /force  
# Verify the configuration  
Get-ItemProperty -Path $RegPath -Name "AllowDomainPINLogon"
```