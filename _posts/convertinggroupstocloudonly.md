---
layout: post
title: "Convert Ad synced groups to cloud only"
---

This blog is about converting AD > Entra group objects to cloud-only objects. Currently, the only supported way to convert synchronized users to cloud-only is to disable directory synchronization on the entire Tenant with Microsoft Graph. [The Microsoft learn article](https://learn.microsoft.com/en-us/microsoft-365/enterprise/turn-off-directory-synchronization?view=o365-worldwide) only references converting user accounts, so it is unclear if other object types like (mail enabled) security groups, contacts and distribution lists are also converted to cloud-only 
