---
# required metadata

title: Create a discovery script for custom compliance policy in Microsoft Intune
description: Create scripts for Linux or Windows devices to discover the settings you define as custom compliance settings for Microsoft Intune.
keywords:
author: brenduns
ms.author: brenduns
manager: dougeby
ms.date: 10/19/2022
ms.topic: conceptual
ms.service: microsoft-intune
ms.subservice: protect
ms.localizationpriority: medium
ms.technology:

# optional metadata

#ROBOTS:
#audience:

ms.reviewer: tycast
ms.suite: ems
search.appverid: MET150
#ms.tgt_pltfrm:
ms.custom: intune-azure
ms.collection: M365-identity-device-management
---

# Custom compliance discovery scripts for Microsoft Intune

Before you can use [custom settings for compliance](../protect/compliance-use-custom-settings.md) with Microsoft Intune, you must define a script for discovery of custom compliance settings on devices. The script you use depends on the platform:

- Linux devices, use a POSIX-compliant shell script
- Windows devices use a PowerShell script

The script deploys to devices as part of your custom compliance policies. When compliance runs, the script discovers the settings that are defined by the JSON file that you also provide through custom compliance policy.

All discovery scripts:

- Are added to Intune before you create a compliance policy. After being added, scripts are available to select when you create a compliance policy with custom settings.
  -   Each discovery script can only be used with one compliance policy, and each compliance policy can only include one discovery script.
  -   Discovery scripts that have been assigned to a compliance policy can't be deleted until the script has been unassigned from the policy.
- Run on a device that receives the compliance policy. The script evaluates the conditions of the JSON file you upload when creating a custom compliance policy.
- Identify one or more settings, as defined in the JSON, and return a list of discovered values for those settings. A single script can be assigned to each policy, and supports discovery of multiple settings.

In addition, the PowerShell script for Windows:

- Must be compressed to output results in a single line. For example: `$hash = @{ ModelName = "Dell"; BiosVersion = "1.24"; TPMChipPresent = $true}`
- Must include the following line at the end of the script: `return $hash | ConvertTo-Json -Compress`

## Limits

The scripts you write must be within the following limits in order to successfully return compliance data to Intune:

- Scripts can be no larger than 1 megabyte (MB) each.
- Output generated by each script can be no larger than 1 MB.
- Scripts must have a limited run time:  
  - On Linux, scripts must take five minutes or less to run.
  - On Windows, scripts must take 10 minutes or less to run.

## Sample discovery script for Windows

The following example is a sample PowerShell script that you would use for Windows devices:

```powershell
$WMI_ComputerSystem = Get-WMIObject -class Win32_ComputerSystem
$WMI_BIOS = Get-WMIObject -class Win32_BIOS 
$TPM = Get-Tpm

$hash = @{ ModelName = $WMI_ComputerSystem.Model; BiosVersion = $WMI_BIOS.SMBIOSBIOSVersion; TPMChipPresent = $TPM.TPMPresent}
return $hash | ConvertTo-Json -Compress
```

The following example is the output of the sample script:

```powershell
PS C:\Users\apervaiz\Documents> .\sample.ps1
{"ModelName":  "Dell","BiosVersion":  1.24,"TPMChipPresent":  true}
```

## Sample discovery script for Linux

> [!NOTE]  
> Discovery scripts in Linux are run in the User's context and as such they cannot check for System level settings that require elevation. An example of this is the state/hash of the `/etc/sudoers` file.

Discovery scripts for Linux must be POSIX-compliant shell scripts, such as Bash. However, the scripts can call more complex interpreters from inside the script, like Python. To successfully use other interpreters, they must be correctly installed and configured on the devices in advance of receiving the discovery script.  

**About POSIX-compliant syntax**: Because the custom compliance script interpreter for Linux supports only a POSIX-compliant shell, it’s important to use POSIX-syntax.

The following are examples of syntax that is compliant vs not compliant:

- Compliant:

  ```Shell
  functionName() {
    // scope of function with compliant syntax
    }
  ```

   For example, `["$a = foo]` - Use of a single equal sign for a string comparison is POSIX-complaint.

- Not compliant:

  ```Shell
  function functionName() {
    // scope of function with non POSIX compliant syntax
    }
  ```

   For example, `["$a == foo]` - Use of a double equal sign for a string comparison isn't POSIX-complaint.

For more information, the following guide might be of use [POSIX Shell Tutorial (grymoire.com)](https://www.grymoire.com/Unix/Sh.html), a third-party website.

## Add a discovery script to Intune

Before deploying your script in production, test it in an isolated environment to ensure the syntax you use behaves as expected.

1. Sign into Microsoft Endpoint Manager admin center and go to  **Endpoint security** > **Device compliance** > **Scripts** > **Add** > *(choose your platform)*.
2. On **Basics**, provide a *Name*.
3. On **Settings**, add your script to *Detection script*. Review your script carefully. Intune doesn’t validate the script for syntax or programmatic errors.
4. ***For Windows only*** - On **Settings**, configure the following behavior for the PowerShell script:

   - **Run this script using the logged on credentials** – By default, the script runs in the System context on the device. Set this value to Yes to have it run in the context of the logged-on user. If the user isn’t logged in, the script defaults back to the System context.
   - **Enforce script signature check** – For more information, see [about_Signing](/powershell/module/microsoft.powershell.core/about/about_signing?view=powershell-7.1&preserve-view=true) in the PowerShell documentation.
   - **Run script in 64 bit PowerShell Host** – By default, the script runs using the 32-bit PowerShell host. Set this value to *Yes* to force the script to run using the 64-bit host instead.

5. Complete the script creation process. The script is now visible in the *Scripts* pane of the Microsoft Endpoint Manager admin center and is available to select when configuring compliance policies.

Also, note that the workflow for uploading these scripts to the Microsoft Endpoint Manager admin center does not support scope tags at this time. You must be targeted with the default scope tag to create, edit, or see custom compliance discovery scripts.

## Next steps

- [Use custom compliance settings](../protect/compliance-use-custom-settings.md)  
- [Create a JSON for custom compliance](../protect/compliance-custom-json.md)
- [Create a compliance policy](../protect/create-compliance-policy.md)
