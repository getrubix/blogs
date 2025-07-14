---
author: steve@getrubix.com
categories:
- intune
- powershell
- azure
date: Wed, 12 May 2021 18:42:22 +0000
description: "Can you smell that? It’s the smell of almost being done deploying SCEP certificates to Windows 10 devices from Intune via the Intune SCEP connector and NDES server. In case you missed it, you can start from Part 1, here. Part 4: Adding the root, deploying SCEP and achieving"
slug: ndes-and-scep-for-intune-part-4
tags:
- endpoint manager
- azure
- configuration profiles
- aad
- script
- powershell
- intune
thumbnail: https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/thumbnails/ndes-and-scep-for-intune-part-4_thumbnail.jpg
title: NDES and SCEP for Intune Part 4
---

Can you smell that?  It’s the smell of almost being done deploying SCEP certificates to Windows 10 devices from Intune via the Intune SCEP connector and NDES server.

_In case you missed it, you can start from Part 1,_ [_here_](https://www.getrubix.com/blog/ndes-and-scep-for-intune-part-1)_._

**Part 4: Adding the root, deploying SCEP and achieving victory**  
&nbsp;

### **Export the Root Certificate (CA)**

Log into the CA and open an elevated CMD prompt. Type the following:

```
certutil -ca.cert C:\root.cer
```

![Picture1.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620844082586-IK38KPUSED1GWDIHAHL1/Picture1.png)

Obviously, feel free to use whatever path you’re comfortable with for the root certificate.

&nbsp;

### **Deploy Trusted Root Certificate Profile (Intune)**

Log into Intune at [https://endpoint.microsoft.com](https://endpoint.microsoft.com) and navigate to:

**Devices → Windows → Configuration profiles → +Create profile → Windows 10 → Templates → Trusted certificate**

![Picture2.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620844124943-PIJ7DKFWOPCW2HFV5MR9/Picture2.png)

In the **Certificate file** field, upload your exported `root.cer`.

Set **Destination store** to:  
**Computer certificate store – Root**

![Picture3.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620844189456-D142NHAIMWHENXMPN2IV/Picture3.png)

Assign this profile to a device group.

&nbsp;

### **Configure SCEP Profile (Intune)**

Still in Endpoint Manager, create another configuration profile:

**Windows 10 → Templates → SCEP certificate**

![Picture4.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620844200395-OE1DZ26TK9HC3B4S2JWK/Picture4.png)

Fill out the fields:

- **Certificate type**: `Device`
- **Subject name format**: `CN={{AAD_Device_ID}}`
- **Certificate validity period**: `2 Years`
- **Key storage provider (KSP)**: `Enroll to TPM KSP if present, otherwise Software KSP`
- **Key usage**: `Key encipherment, Digital signature`
- **Key size**: `2048`
- **Hash algorithm**: `SHA-2`
- **Root Certificate**: *Your Root Cert from previous step*
- **Extended key usage**: `Client Authentication`
- **SCEP Server URLs**: `https://<your-external-url>/certsrv/mscep/mscep.dll`

![Picture5.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620844323147-2ZCUWN7NQJJ3VQZDL8AH/Picture5.png)

Assign the SCEP profile to a device group.

&nbsp;

---

## Epilogue: Troubleshooting

### Validate NDES PowerShell Script

Use Microsoft’s `Validate-NDESConfiguration.ps1` script, found [here](https://github.com/microsoftgraph/powershell-intune-samples/blob/master/CertificationAuthority/Validate-NDESConfiguration.ps1). Run this on your NDES server.

![Picture7.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620844579892-FW45116LXL1UP7X2KE7C/Picture7.png)

&nbsp;

### **Test the Azure App Proxy**

Navigate to the external URL from any browser. You should see the IIS page:

![Picture8.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620844619513-K3Y3WWCBXXRNRRZJUWVV/Picture8.png)

Then, test the full MSCEP path:  
[**https://<yourExternalURL>/certsrv/mscep/mscep.dll**](https://%3cyourExternalURL%3e/certsrv/mscep/mscep.dll)

You should see something like this:

![Picture9.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620844646040-ULJHYIBKV79XQ0RBKEVG/Picture9.png)

This is expected—it’s a service, not a browsable site.

&nbsp;

### **If Using Hybrid Join**

For **Hybrid Azure AD Join** via Autopilot, use this format:

**Subject name format**: `CN={{FullyQualifiedDomainName}}`

![Picture6.png](https://getrubixsitecms.blob.core.windows.net/public-assets/content/v1/5dd365a31aa1fd743bc30b8e/1620844902326-UYI3BTDCUTA6WTGKEHMW/Picture6.png)

&nbsp;

### **Re-trace Your Steps**

If things aren’t working, don’t panic. There are a lot of steps and it’s normal to miss one. Repeating the process helps build familiarity.

Reach out with questions: [steve@getrubix.com](mailto:steve@getrubix.com)

Thanks for reading!
