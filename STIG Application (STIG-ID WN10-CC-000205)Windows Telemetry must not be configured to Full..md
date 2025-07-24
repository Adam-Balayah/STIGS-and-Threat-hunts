```
<#
.SYNOPSIS
    This script ensures that Windows 10 telemetry is not set to 'Full', reducing the risk of sensitive data being sent to Microsoft.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-CC-000205

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5
#>

# Define registry path  
$RegPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection"  
# Ensure the registry path exists  
If (!(Test-Path $RegPath)) {  
New-Item -Path $RegPath -Force  
}  
# Set AllowTelemetry to 0 (Security) to restrict diagnostic data collection  
Set-ItemProperty -Path $RegPath -Name "AllowTelemetry" -Value 0 -Type DWord  
# Force Group Policy update to apply changes  
gpupdate /force  
# Verify the configuration  
Get-ItemProperty -Path $RegPath -Name "AllowTelemetry"
```