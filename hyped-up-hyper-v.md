---
author: steve@getrubix.com
date: Tue, 17 Nov 2020 03:42:07 +0000
description: '"Time for something a bit different.&nbsp; This blog is about, well,
  another blog.&nbsp; Specifically, Oliver Kieselbach''s recent write up on managing
  a Hyper-V lab for Intune.https://oliverkieselbach.com/2020/11/13/working-with-hyper-v-vms-in-an-intune-lab-environment/If
  you haven''t read it yet, than stop right now and get to it.&nbsp; Oliver Kieselbach
  is a Microsoft MVP and has a"'
slug: hyped-up-hyper-v
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/hyped-up-hyper-v_thumbnail.jpg
title: Hyped-up Hyper-V
---

Time for something a bit different.  This blog is about, well, another blog.  Specifically, Oliver Kieselbach's recent write up on managing a Hyper-V lab for Intune.

[https://oliverkieselbach.com/2020/11/13/working-with-hyper-v-vms-in-an-intune-lab-environment/](https://oliverkieselbach.com/2020/11/13/working-with-hyper-v-vms-in-an-intune-lab-environment/)

If you haven't read it yet, than stop right now and get to it.  Oliver Kieselbach is a Microsoft MVP and has a knack for coming up with all kinds of clever scripts and components that make Intune a better place. 

One of the first things I did after reading was apply Mr. Kieselbach's script to my own Hyper-V environment, with a few modifications.  Instead of building a new VHD every time, I clone a sys-prepped image of Windows 10, depending on the build number.  Here is the modified version:

```
<#
Version: 1.0
Author: Oliver Kieselbach (oliverkieselbach.com)
Script: Create-MyVM.ps1
Description:
The script crates a VM on a Hyper-V host with TPM and starts it including the VMConnect client. 
Release notes:
Version 1.0: Original published version. 
The script is provided "AS IS" with no warranties.

#>
<# Modification: This version will prompt for the Windows build number and clone a Gold-Master image disk to build the new VM
#>

$VMName = Read-Host -Prompt 'Enter VM name'
$version = Read-Host -Prompt 'Enter Windows build'
if (($CPUCount = Read-Host -Prompt "CPU count? [default=2, Enter]") -eq "") { $CPUCount = 4 } 

Copy-Item -Path "F:\Templates\GM-$($version).vhdx" -Destination "F:\Hyper-V\Virtual hard disks\$($VMName).vhdx" -Force | Out-Null
 
# some definitions for Network and VM storage path
$VMSwitchName = "VNET"
$VhdxPath = "F:\Hyper-V\Virtual hard disks\$VMName.vhdx"
$VMPath = "F:\Hyper-V\Virtual machines\Virtual Machines"
 
# I'm not usign Enhanced Session Mode, so we can run this once to disable it on the host
#Set-VMHost -EnableEnhancedSessionMode $false
 
New-VM -Name $VMName -BootDevice VHD -VHDPath $VhdxPath -Path $VMPath -Generation 2 -Switch $VMSwitchName
Set-VM -VMName $VMName -ProcessorCount $CPUCount
Set-VMMemory -VMName $VMName -StartupBytes 4GB -DynamicMemoryEnabled $false
Set-VMSecurity -VMName $VMName -VirtualizationBasedSecurityOptOut $false
Set-VMKeyProtector -VMName $VMName -NewLocalKeyProtector
Enable-VMTPM -VMName $VMName
Enable-VMIntegrationService -VMName $VMName -Name "Guest Service Interface"
```

Now of course, you'll notice a few other tweaks; no dynamic memory enabled, no auto-start- fairly minor changes.  The big difference comes in the line that clones the disk:

```
Copy-Item -Path "F:\Templates\GM-$($version).vhdx" -Destination "F:\Hyper-V\Virtual hard disks\$($VMName).vhdx" -Force | Out-Null
```

The **$version** variable comes from the initial prompt, which becomes **GM-$($version)**.  In my case, there are three images: GM-1909, GM-2004, and GM-20H2. 

![Untitled picture.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1605583898044-20FY2K0R1B7ATQE1599A/Untitled+picture.png)

![2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1605583937392-JN0HD6IPKINOCFFX60JF/2.png)

As you can see in the line above, I use the $VMName variable to match the name of the cloned disk to the VM itself. 

Why use the image at all?  Because most of the time, when I'm building a VM it is for Autopilot enrollment testing.  So naturally, one of the first things I need to do is register the VM in my Azure lab.  Instead of running the same PowerShell commands every time, I simply download everything I need so it's sitting on the VM:

```
Install-PackageProvider -Name Nuget -confirm:$false -force
Install-Script -Name Get-WindowsAutopilotinfo -confirm:$false -force
Install-Script -Name Get-AutopilotDiagnostics -confirm:$false -force
Install-Module WindowsAutopilotIntune -confirm:$false -force
```

You're probably familiar with the first two up there.  They're needed for generating the device ID hardware hash.  I also pull down Michael Niehaus's **Get-AutopilotDiagnostics** script so I can quickly troubleshoot an enrollment gone bad.

Finally, the **WindowsAutopilotIntune** module allows me to use the "_\-Online_" switch in Get-WindowsAutopilotInfo.  Instead of generating a hardware hash .CSV, I can just authenticate into Azure and post the device directly to Autopilot.

So when I boot one of my fresh, new VMs, all I need to do is bring up PowerShell and use:

```
Set-ExecutionPolicy Bypass -Force 
```

```
Get-WindowsAutopilotInfo -Online
```

![3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1605584065212-SC1969Q1TLUN2MKY9YVA/3.png)

![4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1605584091337-N75F82KSCBW6QSMXSPDS/4.png)

It's a great setup.  I can simply remote into my Hyper-V server from any of the PCs in my house and create as many VMs as I need, each one ready for Autopilot.

Again, huge thanks to Oliver Kieselbach for the PowerShell script.
