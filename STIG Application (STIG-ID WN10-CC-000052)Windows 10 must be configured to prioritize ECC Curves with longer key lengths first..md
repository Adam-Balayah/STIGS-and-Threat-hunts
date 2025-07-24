```
<#
.SYNOPSIS
    This script ensures that Windows 10 is configured to prioritize ECC curves with longer key lengths (e.g., NistP384 and NistP256) for SSL, enhancing cryptographic security.

.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-CC-000052

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5
#>

# Define registry path  
$RegPath = "HKLM:\SOFTWARE\Policies\Microsoft\Cryptography\Configuration\SSL\00010002"  
# Ensure the registry path exists  
If (!(Test-Path $RegPath)) {  
New-Item -Path $RegPath -Force  
}  
# Set EccCurves to NistP384 and NistP256 in the correct order  
$EccCurves = "NistP384", "NistP256"  
Set-ItemProperty -Path $RegPath -Name "EccCurves" -Value $EccCurves -Type MultiString  
# Force Group Policy update to apply changes  
gpupdate /force  
# Verify the configuration  
Get-ItemProperty -Path $RegPath -Name "EccCurves"
```