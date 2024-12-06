---
layout: post
title: "Access AD DS integrated shares on Azure Files with Windows Hello for Business Hybrid"
---
Welcome to my first blog. Azure files can be used as an alternative to on-premises file servers. Just like on-premise shares, they can be integrated with Active Directory Domain Services (AD DS). When using Microsoft Entra joined windows devices that are managed with Intune, Windows Hello for Business provides passwordless sign-in (PIN and/or Biometrics) to the device.  Windows Hello for business hybrid can be used additionaly to provide single sign-on from the cloud-only device to Active Directory integrated resources. In this post I want to share how I've set up this functionality in my lab environment, to provide single sign-on to Azure Files.

Following components are part of this lab setup:
- Active Directory domain with Microsoft Entra Kerberos (AzureADKerberos) object on the on-premises domain
- A user that is synchronized from Active Directory to Microsoft Entra
- Intune policy that enables Cloud Trust for on-premises authentication
- Intune policy that enforces WHFB on devices
- Azure subscription with a storage account for hosting the Azure Files share
- Entra Joined Windows 11 virtual machine in Hyper-V with Windows Hello for business (WHFB) enabled

### Step 1: Enable Microsoft Entra Kerberos
Follow [these steps](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust?tabs=intune) to enable the Microsoft Entra Kerberos for your Active Directory domain. This creates the AzureADKerberos object in the domain controllers OU.
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture1.png" alt="image" style="float: left; margin-right: 10px;" />

### Step 2: Configure Windows Hello for Business policy that enabled Cloud Trust for on-premises authentication
In the above Learn article the steps for configuring the Intune configuration policy are also described. I created the following policy and applied it to all users:
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture2.png" alt="image" style="float: left; margin-right: 10px;" />

### Step 3: Enable Windows Hello For business
If not already enabled, make sure that Windows Hello for Business is enabled for the device you want to test with. I created a policy for all users in the Devices > Enrollment > Windows Hello for Business section but you can also target specific users with an Identity protection settings catalog policy. 

### Step 4: Create a storage account in Azure 
I created a storage account in my trial Azure subscription:
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture4.png" alt="image" style="float: left; margin-right: 10px;" />

In the file shares section I created a file share:
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture5.png" alt="image" style="float: left; margin-right: 10px;" />

In order to integrate the file share with Active Directory, [the storage account needs to be registered in your on-premises AD.](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-identity-ad-ds-enable)   
After running these steps, a computer account is created in the local active directory that represents the storage account: 
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture6.png" alt="image" style="float: left; margin-right: 10px;" />

When you go back to the properties of the file share in Azure, it will now show that the directory service is configured:
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture7.png" alt="image" style="float: left; margin-right: 10px;" />

When you click on ”configured” it shows that AD DS is enabled:
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture9.png" alt="image" style="float: left; margin-right: 10px;" />

I also configured the share level permissions and gave my synchronized account (test.sync) the Storage File Data SMB Share Elevated Contributor permissions:
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture10.png" alt="image" style="float: left; margin-right: 10px;" />

### Step 5: Access the AD share from a Entra joined W11 device 
I created a Windows 11 VM that is Entra Joined (corporate owned). In the VM settings I enabled TPM, this is required for installing W11. In the network settings of the VM, I set the DNS server to the IP of my domain controller, this way the VM has a line of sight to the domain controller. After enabling the Windows Hello for business setting (step 3) the machine was prompted for registration of Windows Hello for business (PIN sign in). At first the registration didn’t succeed because of error 0x80090010. I looked up the error on https://aka.ms/pinerrors but this didn't help me much in finding a cause. After some searching I tried disabling the Enhanced session mode policy for both server and user in Hyper-V, after this PIN registration worked without issues. 
 
I signed in with my PIN on my Entra Joined device:
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture12.png" alt="image" style="float: left; margin-right: 10px;" />

After running the klist command, no Kerberos tickets showed up. This is expected because we haven’t yet contacted a domain controller for requesting access to a file share. 
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture13.png" alt="image" style="float: left; margin-right: 10px;" />

For mounting the file share, I ran the command net use Z: \\%storateaccountname%\file.core.windows.net\%sharename% 
The share is now mounted in file explorer without requiring further authentication:
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture14.png" alt="image" style="float: left; margin-right: 10px;" />

I am able to create new folders and files:
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture15.png" alt="image" style="float: left; margin-right: 10px;" />

If we run the klist command again, we can now see two kerberos tickets have been handed out by the Cloud Trust mechanism:
<img src="https://matthijstuenter.github.io/assets/img/2024-12-04/Picture16.png" alt="image" style="float: left; margin-right: 10px;" />


