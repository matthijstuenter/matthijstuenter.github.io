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
Follow these steps https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust?tabs=intune to enable the Microsoft Entra Kerberos for your Active Directory domain. This creates the AzureADKerberos object in the domain controllers OU.

![image](https://github.com/user-attachments/assets/06110f6a-9524-459e-bc2b-705dfbf8c188)

### Step 2: Configure Windows Hello for Business policy that enabled Cloud Trust for on-premises authentication
In the above Learn article the steps for configuring the Intune configuration policy are also described. I created the following policy and applied it to all users:

![image](https://github.com/user-attachments/assets/c838837c-996d-41ba-a1a4-ac5046db8110)

### Step 3: Enable Windows Hello For business
If not already enabled, make sure that Windows Hello for Business is enabled for the device you want to test with. I created a policy for all users in the Devices > Enrollment > Windows Hello for Business section but you can also target specific users with an Identity protection settings catalog policy. 

### Step 4: Create a storage account in Azure 
I created a storage account in my trial Azure subscription:

![image](https://github.com/user-attachments/assets/8e86a0db-df0d-446c-93e6-77162f2e73ac)

In the file shares section I created a file share:

![image](https://github.com/user-attachments/assets/bb9add12-836e-4238-b196-c50ce5c4a0f9)

In order to integrate the file share with Active Directory, the storage account needs to be registered in your on-premises AD. https://learn.microsoft.com/en-us/azure/storage/files/storage-files-identity-ad-ds-enable   
After running these steps, a computer account is created in the local active directory that represents the storage account: 

![image](https://github.com/user-attachments/assets/c2ab4478-0cf4-4f82-a16d-b413bab8865a)

When you go back to the properties of the file share in Azure, it will now show that the directory service is configured:

![image](https://github.com/user-attachments/assets/cfa19c26-c347-4fa5-a87d-182496e38cd8)

When you click on ”configured” it shows that AD DS is enabled:

![image](https://github.com/user-attachments/assets/5e7d8bdc-8e12-4315-9735-de83d40b807e)

I also configured the share level permissions and gave my synchronized account (test.sync) the Storage File Data SMB Share Elevated Contributor permissions:

![image](https://github.com/user-attachments/assets/89041044-ebae-456e-9f42-fe0e486f75b2)

### Step 5: Access the AD share from a Entra joined W11 device 
I created a Windows 11 VM that is Entra Joined (corporate owned). In the VM settings I enabled TPM, this is required for installing W11. In the network settings of the VM, I set the DNS server to the IP of my domain controller, this way the VM has a line of sight to the domain controller. After enabling the Windows Hello for business setting (step 3) the machine was prompted for registration of Windows Hello for business (PIN sign in). At first the registration didn’t succeed because of error 0x80090010. I looked up the error on https://aka.ms/pinerrors but this didn't help me much in finding a cause. After some searching I tried disabling the Enhanced session mode policy for both server and user, after this PIN registration worked without issues. 

![image](https://github.com/user-attachments/assets/03065116-5073-4243-9d95-eaaa7e92ca2d)
 
I signed in with my PIN on my Entra Joined device:

![image](https://github.com/user-attachments/assets/c8676df9-ac76-41e0-b925-ff46a84319da)

After running the klist command, no Kerberos tickets showed up. This is expected because we haven’t yet contacted a domain controller for requesting access to a file share. 
 
For mounting the file share, I ran the command net use Z: \\%storateaccountname%\file.core.windows.net\%sharename% 
The share is now mounted in file explorer without requiring further authentication:

![image](https://github.com/user-attachments/assets/6763101d-d7ad-417f-83bb-260a9a5ecb05)

I am able to create new folders and files:

![image](https://github.com/user-attachments/assets/d46e85c6-c96f-49a5-bd00-7e3533637548)

If we run the klist command again, we can now see two kerberos tickets have been handed out by the Cloud Trust mechanism:

![image](https://github.com/user-attachments/assets/515e0d37-ce3a-4eba-b41c-84c2e536a92c)


