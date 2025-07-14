---
author: GuestUser
date: Wed, 27 Mar 2024 17:48:07 +0000
description: '"Every now and then I either need to look for a conflicting group policy,
  or some other software setting that is sitting in the registry.A good example of
  conflicting GPO that others have blogged about – Windows Update policies. Generally
  when deploying Update Rings and Feature Updates through"'
slug: looking-for-something-use-a-remediation-GXD0c
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/looking-for-something-use-a-remediation-GXD0c_thumbnail.jpg
title: Looking for something Use a remediation
---

Every now and then I either need to look for a conflicting group policy, or some other software setting that is sitting in the registry.

A good example of conflicting GPO that others have blogged about – Windows Update policies. Generally when deploying Update Rings and Feature Updates through Intune, you don’t want any conflicting settings hitting HKLM:\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate (Intune alone does NOT write anything to this register path – if you are co-managed with MECM then some settings here are normal, depending).

So, if I’m deploying these policies for the first time, and I’m either having issues or unsure of whether any conflicts are present, I could just check this registry path remotely through Intune. Here’s the detection script:

$registry = reg.exe query "HKLM\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate" | ConvertFrom-String | Select P2, P4 | ConvertTo-Csv -NoTypeInformation | Select-Object -Skip 1

$final = \[string\]::join(",",($registry.Split("\`n")))
Write-Host $final 

I know that first line is some funky formatting; I was struggling to decide on how I wanted it to show in the console. There’s probably a better way, but this will at least show everything in the single write-host for the pre-remediation output. Also, you may want to peak at the sub-key “AU” as well under WindowsUpdate, which you could do as a separate remediation or combine somehow with the above.

Even if your device is not domain joined, perhaps you have a third-party RMM tool that is unknowingly setting these values? Here’s another one I’ve used to see if workloads/policies are working correctly with “Windows Update” as the default service provider:

$MUSM = New-Object -ComObject "Microsoft.Update.ServiceManager"
$list = $MUSM.Services | select Name, IsDefaultAUService

foreach($service in $list)
{
    if($service.IsDefaultAUService -eq "True")
    {
        Write-Host "$($service.Name) is default AU Service"
        exit 0
    }
}

Write-Host "Error - could not determine default AU Service"
exit 0 

The big thing to note here is that these are only detection scripts – I am not uploading actual remediation scripts with these. Perhaps one day if I removed GPO and/or the MECM client from these devices, and if some of these entries did not get cleaned up, then I certainly could add a remediation script to delete those paths.

Here’s a separate example… folks looking to monitor usage of Windows Hello for Business vs. Convenience PIN. You can run this detection-only script as the logged on user:

$loggedOnUserSID = (New-Object System.Security.Principal.NTAccount($env:username)).Translate(\[System.Security.Principal.SecurityIdentifier\]).value
$value = Get-ItemProperty "HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Authentication\\Credential Providers\\{D6886603-9D2F-4EB2-B667-1971041FA96B}\\$loggedOnUserSID" -ErrorAction SilentlyContinue | Select -ExpandProperty "LogonCredsAvailable"
 
$Dsregcmd = New-Object PSObject ; Dsregcmd /status | Where {$\_ -match ' : '} | ForEach {$Item = $\_.Trim() -split '\\s:\\s'
$Dsregcmd | Add-Member -MemberType NoteProperty -Name $($Item\[0\] -replace '\[:\\s\]','') -Value $Item\[1\] -EA SilentlyContinue}
$DsregcmdNgcSet = $Dsregcmd.NgcSet
 
if(($value -eq 1) -and ($DsregcmdNgcSet -eq "NO"))
{
    Write-Host "Convenience PIN detected."
    exit 1
}
elseif($DsregcmdNgcSet -eq "YES")
{
    Write-Host "Windows Hello for Business key detected."
    exit 0
}
else
{
    Write-Host "NO Convenience PIN or WHfB Key detected."
    exit 0
}

I think some folks have had issues transitioning from Convenience PINs to WHfB Keys, so this can help identify devices that need the PIN removed first.

Anyway, there’s tons of other use cases for searching/reviewing your devices’ registry entries - I would suggest reviewing the other blogs out there that have both reporting and actual remediation examples (I especially like the ones that upload custom logs to log analytics workspaces). Enjoy!
