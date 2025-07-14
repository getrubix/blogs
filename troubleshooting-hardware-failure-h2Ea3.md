---
author: GuestUser
date: Sun, 31 Jan 2021 21:19:22 +0000
description: '"One common policy that can encounter issues related to hardware is
  Bitlocker encryption. When looking at the per-device status for the policy, you
  can click on one of the failing devices to view the device configuration summary.
  If Bitlocker is failing, select the policy and see which components"'
slug: troubleshooting-hardware-failure-h2Ea3
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/troubleshooting-hardware-failure-h2Ea3_thumbnail.jpg
title: Troubleshooting Hardware Failure
---

One common policy that can encounter issues related to hardware is **Bitlocker encryption**. When looking at the _per-device status_ for the policy, you can click on one of the failing devices to view the **device configuration** summary. If Bitlocker is failing, select the policy and see which components have an error – the image below shows a typical failure where the encryption does not successfully complete:

![bitlocker.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1612127514873-6DCTLQ382LI0TK3CSA91/bitlocker.png)

The BitLocker policy is supposed to automatically and silently enables BitLocker on a device - that means that BitLocker should enable successfully without presenting any UI to the end user, even when that user isn't a local Administrator on the device.

To understand the failure, let’s first look at the requirements for BitLocker to function correctly. A device must meet the following conditions to be eligible for silently enabling BitLocker:

-   If end users log in to the devices as Administrators, the device must run Windows 10 version **1803 or later**.
    
-   If end users log in to the the devices as Standard Users, the device must run Windows 10 version **1809 or later**.
    
-   The device must be Azure AD Joined
    
-   Device must contain **TPM (Trusted Platform Module) 2.0**. If configuring a Virtual Machine, some platforms may be able
    
-   The BIOS mode must be set to Native UEFI only.
    

The following two settings must be configured in the BitLocker policy:

-   Warning for other disk encryption = **Block**
    
-   Allow standard users to enable encryption during Azure AD Join = **Allow**
    

The BitLocker policy must not require use of a startup PIN or startup key. When a TPM startup PIN or startup key is required, BitLocker can't silently enable and requires interaction from the end user. This requirement is met through the following three BitLocker OS drive settings in the same policy:

-   Compatible TPM startup PIN must not be set to Require startup PIN with TPM
    
-   Compatible TPM startup key must not set to Require startup key with TPM
    
-   Compatible TPM startup key and PIN must not set to Require startup key and PIN with TPM
    

Additional Logs
---------------

If any of the requirements are not met, or if any of the policies are set incorrectly, you will typically see the error message in the screenshot above. For devices that should be meeting the requirements, you can collect additional logs from Event Viewer and the MdmDiagnosticsTool.exe output.

From Event Viewer, navigate to **Applications & Services Logs -> Microsoft -> Windows -> BitLocker-API -> Management.** The logs might contain the following events – click on each event to learn more about the issue:

-   [Event ID 853: Error: A compatible Trusted Platform Module (TPM) Security Device cannot be found on this computer](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/ts-bitlocker-intune-issues#issue-1)
    
-   [Event ID 853: Error: BitLocker Drive Encryption detected bootable media (CD or DVD) in the computer](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/ts-bitlocker-intune-issues#issue-2)
    
-   [Event ID 854: WinRE is not configured](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/ts-bitlocker-intune-issues#issue-3)
    
-   [Event ID 851: Contact manufacturer for BIOS upgrade](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/ts-bitlocker-intune-issues#issue-4)
    
-   [Error message: The UEFI variable 'SecureBoot' could not be read](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/ts-bitlocker-intune-issues#issue-6)
    
-   [Event ID 846, 778, and 851: Error 0x80072f9a](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/ts-bitlocker-intune-issues#issue-7)
    
-   [Error message: Conflicting Group Policy settings for recovery options on operating system drives](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/ts-bitlocker-intune-issues#issue-5)
    

For MdmDiagnosticsTool.exe, you can view the _TpmHliInfo\_Output.txt_ for TPM, which contains basic details about the TPM in the device (this includes the manufacturer, the firmware level of that TPM, whether it has a required EK cert, and more). Other Event Viewer data from the tool is summarized on this article: [https://oofhours.com/2019/10/08/troubleshooting-windows-autopilot-a-reference/](https://oofhours.com/2019/10/08/troubleshooting-windows-autopilot-a-reference/). For other policies outside of BitLocker, the collected Event Viewer logs may help to reveal other underlying issues with deployment failures.
