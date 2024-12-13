Please reference this KBA for pictured instructions: [How to deploy Cisco Secure Client via Intune (macOS)](https://support.umbrella.com/hc/en-us/articles/32940834196884-How-to-deploy-Cisco-Secure-Client-via-Intune-macOS)


**Bash Shell script to deploy Cisco Secure Client 5 to MacOS devices via Intune**

This is a step-by-step guide on how to deploy Cisco Secure Client to your macOS devices via Intune. It is done with the assumption that your macOS device is already Managed by Intune. If you need to MDM your macOS device, please follow Microsoft's documentation. 

**DISCLAIMER: This article is provided as-is as of 02-01-2024. Umbrella support does not guarantee these instructions will remain valid after this date and is subject to change based on updates from Microsoft Intune and Apple macOS.**




**Procedure**

1) Download the Cisco Secure Client disk image (DMG) from your Umbrella dashboard, under Deployments --> Roaming Computers --> top right: Roaming Client --> Pre-Deployment Package --> macOS.
2) Follow our [documentation](https://docs.umbrella.com/umbrella-user-guide/docs/customize-macos-installation-of-cisco-secure-client) to craft your disk image (DMG) until you've reached Step 5. This is done so you can create the install_choices.xml to enable/disable the modules you need for your environment, with ACTransforms.xml being optional to hide the VPN module UI.
3) Eject the disk image you've just crafted. One way to verify the disk image has been crafted properly is by double-clicking "csc-writeable.dmg" and then verify ACTransforms.xml, install_choices.xml, and OrgInfo.json are still in it's respective place. Place this disk image in a central location that your macOS devices can access. Your NAS or shared URL should contain the DMG file. You would need to drag this file from the above disk image (DMG) into your NAS or shared folder.

**Deployment Options for Cisco Secure Client on macOS via Intune:**
1) MacOS app (DMG):

Important note: The DMG app option is not supported in Intune for macOS deployments. DMG files are commonly used for macOS applications, but they are not packaged in a format that Intune directly supports for app deployment. Inside a DMG file, the application is not in the .app format directly. You may notice the following error under Installation Status: Failed. This error occurs because the DMG contains a .pkg file instead of an .app file. Therefore, you can proceed to the next option, macOS app (PKG).

2) MacOS app (PKG):

The macOS app (PKG) deployment method in Intune is used to distribute and install the .pkg file on macOS devices. 

Important note: This deployment method will install ALL modules included in the package by default. Therefore, you can use the third option Deploying via Script (Custom Script), which involves using a script in Intune to customize the installation process, allowing you to control which modules are installed during deployment. 

**3) Deploying via Script (Custom Script)**

Using a custom script, we'll utilize a Bash script to execute the installation command for the Cisco Secure Client with the Umbrella module, including the accepted parameters from the install_choices.xml file.Download the bash shell script from this GitHub Repo. Modify line 6 and line 10 to the respective version you're deploying. On line 12, modify the file path to point directly to the disk image (DMG) file.

In Intune, on the far left menu, navigate to **Devices --> macOS --> Shell scripts --> Add**. Give it a unique name. Upload the bash shell script downloaded from the previous step and ensure the following values are configured:

* Run script as signed-in user: **No**
* Hide script notification on devices: **Yes**
* Script frequency: **Every 15 minutes**
* Max number of times to retry if script fails: **3 times**

In Assignments, select your desired user/device assignment and click Create.


**How to Configure a Silent Install for System Extensions and ManagedLoginItems:**

Next, we'll need to configure and allow Cisco Secure Client's required System Extensions in order for Cisco Secure Client with Umbrella module to run correctly without user interactions. This step in combination with configuring **ManagedLoginItems** will allow Cisco Secure Client with Umbrella module to launch upon device startup.

[Cisco Secure Client Changes Related to macOS 11 (And Later)](https://www.cisco.com/c/en/us/td/docs/security/vpn_client/anyconnect/Cisco-Secure-Client-5/admin/guide/b-cisco-secure-client-admin-guide-5-1/macos11-on-ac.html#Cisco_Reference.dita_129105c0-2c8f-4635-9f2e-89d769ded6d4)




On the far left menu, browse to **Devices --> macOS --> Configuration profiles** and create the following policies to silently enable the required System Extensions in order for Cisco Secure Client with Umbrella module to run correctly without user interactions:

**Create --> New Policy** --> Profile type: **Settings catalog** --> Name: **ManagedLoginItems** --> Add Settings --> search for Managed Login Items --> select **Login > Service Management - Managed Login Items** --> expand the rules at the bottom and select **Comment, Rule Type, and Rule Value.**

Close the right side panel and click + **Edit Instance** and enter the following values, removing the last row for "Team Identifiers":


* Comment: **Cisco Secure Client**
* Rule Type: **Bundle Identifier Prefix**
* Rule Value: **com.cisco.secureclient**
* Team Identifier: **DE8Y96K9QP**

Then, we'll be using the MDM configuration profile to load both the Cisco Secure Client system and the kernel extensions, including the system extension's content filter component.

Navigate to [Cisco Secure Client Changes Related to macOS 11 (And Later)](https://www.cisco.com/c/en/us/td/docs/security/vpn_client/anyconnect/Cisco-Secure-Client-5/admin/guide/b-cisco-secure-client-admin-guide-5-1/macos11-on-ac.html#Cisco_Reference.dita_129105c0-2c8f-4635-9f2e-89d769ded6d4) and copy the sample XML.

Navigate back to **Configuration profiles --> Create --> New Policy** --> Profile type: **Templates --> Custom --> Create** --> Name: **System Extensions**--> Custom Configuration profile name: **System Extensions** --> Deployment channel: **Device channel**. Upload the file you saved from the previous step:

In Assignments, select your desired user/device assignment and click Create.

Then, On the far left menu, navigate to Devices --> macOS and click on one of your macOS device(s) that is suppose receive the Cisco Secure Client deployment. In Overview, click Sync so your desired macOS device(s) can pick up the changes. You may track the progress by navigating to Device configuration on the same page and see if the extensions we've configured has been pushed successfully.

**Deploying the Cisco Umbrella Root Certificate:**

**This step only applies to new deployments of Cisco Secure Client or devices that does not have the Cisco Umbrella Root Certificate deployed previously.** If you're migrating over from the Umbrella Roaming Client or Cisco AnyConnect 4.10 client, and/or have deployed the Cisco Umbrella Root Certificate already in the past, you may skip this step.

In your Umbrella dashboard, under **Deployments --> Configuration --> Root Certificate**, download the Cisco Umbrella Root Certificate.
In Intune, on the far left menu, navigate to **Devices --> macOS --> Configuration profiles --> Create --> New Policy --> Profile type: Templates --> Trusted certificate --> Create.**
Give it a unique name like "Cisco Umbrella Root Certificate". In Configuration settings, upload the root certificate downloaded from the previous step.
In Assignments, select your desired user/device assignment and click Create.
Navigate back to the overview of your macOS devices and click Sync. Just like previously in Step 18, this allows the desired macOS device(s) to pick up the changes during the device's next check-in with Intune.

**Verify:**
You may verify Cisco Secure Client with Umbrella module is working by either browsing to https://policy-debug.checkumbrella.com or by running the following command:

```dig txt debug.opendns.com```

Either output should contain unique and relevant information to your Umbrella organization such as your OrgID.



