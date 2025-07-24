```
<#
.SYNOPSIS
This script ensures that the required U.S. Government legal notice is displayed before console logon on Windows 10 systems.


.NOTES
    Author          : Adam Balayah
    LinkedIn        : 
    GitHub          : 
    Date Created    : 7/20/2025
    Last Modified   : 7/21/2025
    Version         : 1.0
    CVEs            : N/A
    Plugin IDs      : N/A
    STIG-ID         : WN10-SO-000075

.TESTED ON
    Date(s) Tested  : 7/21/2025
    Tested By       : Adam Balayah
    Systems Tested  : STIG-VM17
    PowerShell Ver. : 5

#>

# --- Variables ---
$regPath         = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System'
$captionName     = 'LegalNoticeCaption'
$messageName     = 'LegalNoticeText'

# STIG doesn’t specify a caption, but Windows requires one to show the banner.
# Use the standard WARNING! caption per WN10-SO-000074.
$captionValue    = 'WARNING!'
$messageValue    = @"
You are accessing a U.S. Government (USG) Information System (IS) that is provided for USG-authorized use only.

By using this IS (which includes any device attached to this IS), you consent to the following conditions:

-The USG routinely intercepts and monitors communications on this IS for purposes including, but not limited to, penetration testing, COMSEC monitoring, network operations and defense, personnel misconduct (PM), law enforcement (LE), and counterintelligence (CI) investigations.

-At any time, the USG may inspect and seize data stored on this IS.

-Communications using, or data stored on, this IS are not private, are subject to routine monitoring, interception, and search, and may be disclosed or used for any USG-authorized purpose.

-This IS includes security measures (e.g., authentication and access controls) to protect USG interests—not for your personal benefit or privacy.

-Notwithstanding the above, using this IS does not constitute consent to PM, LE or CI investigative searching or monitoring of the content of privileged communications, or work product, related to personal representation or services by attorneys, psychotherapists, or clergy, and their assistants. Such communications and work product are private and confidential. See User Agreement for details.
"@

# --- 1. Apply Caption & Text ---
Try {
    # Caption
    New-ItemProperty -Path $regPath `
                     -Name $captionName `
                     -Value $captionValue `
                     -PropertyType String `
                     -Force -ErrorAction Stop
    # Text
    New-ItemProperty -Path $regPath `
                     -Name $messageName `
                     -Value $messageValue `
                     -PropertyType String `
                     -Force -ErrorAction Stop

    Write-Host "✔ Successfully set interactive logon caption and text." -ForegroundColor Green
}
Catch {
    Write-Error "✖ Failed to set legal notice: $_"
    exit 1
}

# --- 2. Refresh Group Policy ---
Try {
    gpupdate /force | Out-Null
    Write-Host "✔ Group Policy refreshed." -ForegroundColor Green
}
Catch {
    Write-Warning "⚠ Could not refresh Group Policy automatically."
}

# --- 3. Reboot Note ---
Write-Host "ℹ No reboot required; banner will appear at next logon." -ForegroundColor Yellow

# --- 4. Verification Function ---
function Test-RegistryPolicy {
    param(
        [Parameter(Mandatory)][string]$Path,
        [Parameter(Mandatory)][string]$Name,
        [Parameter(Mandatory)][ValidateNotNullOrEmpty()][string]$ExpectedValue
    )
    try {
        $actual = Get-ItemProperty -Path $Path -Name $Name -ErrorAction Stop | Select-Object -ExpandProperty $Name
        if ($actual -eq $ExpectedValue) {
            Write-Host "PASS: Registry `$Name` at `$Path` matches expected value." -ForegroundColor Green
        }
        else {
            Write-Host "FAIL: Registry `$Name` at `$Path` has value:`n$actual`n(expected:`n$ExpectedValue)" -ForegroundColor Red
        }
    }
    catch {
        Write-Host "FAIL: Unable to read registry `$Path\$Name`." -ForegroundColor Red
    }
}

# --- 5. Registry Verification ---
Test-RegistryPolicy `
    -Path $regPath `
    -Name $captionName `
    -ExpectedValue $captionValue

Test-RegistryPolicy `
    -Path $regPath `
    -Name $messageName `
    -ExpectedValue $messageValue

```