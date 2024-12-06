---
layout: post
title: "Convert Ad synced groups to cloud only"
---

This blog is about converting AD > Entra synchronized group objects to cloud-only objects. Currently, the only supported way to convert synchronized users to cloud-only is to disable directory synchronization on the entire Tenant with Microsoft Graph. [The Microsoft learn article](https://learn.microsoft.com/en-us/microsoft-365/enterprise/turn-off-directory-synchronization?view=o365-worldwide) only references converting user accounts, so for me it's unclear if other object types like (mail enabled) security groups, contacts and distribution lists are also converted to cloud-only objects after disabling directory synchronization. 

Distribution list would need to be recreated... 

There are 5 types of synchronized objects in my exchange hybrid environment:
-A remote (M365) mailbox TestRemoteMailbox@tcompany.myo365.site 
-A user that doesn not have a remote mailbox enabled in Exchange Hybrid called test.sync
-A distribution group TestDistributionGroup@tcompany.myo365.site with the above remote mailbox as a member
-A mail enabled security group TestMailEnabledSecurityGroup@tcompany.myo365.site with the remote mailbox as a member
-A "normal" security group created in AD, with the remote mailbox and a non-remote mailbox user called "test.sync"
-A contact called "testcontact@gmail.com"

Exchange hybrid is configured in the Entra connect sync settings.

I ran the Microsoft Graph command on 20:45 PM




