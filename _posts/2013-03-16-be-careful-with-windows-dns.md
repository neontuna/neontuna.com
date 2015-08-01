---
layout: post
title: Be careful with Windows DNS
date: 2013-03-06 11:15:57
summary: Aging/Savaging can get you into trouble
category: blog
categories: windows admin 
---

Earlier today I was looking into an issue for a client. In an audit report for folder moves/deletes it was shown that a user (Edgar) moved a folder, but did it from a machine that wasn't his (Hank's PC). It turns out this is because both Edgar and Hank had connected to the client's VPN. This VPN uses its own pool of IPs instead of the Windows DHCP server onsite. Therefore the DNS records for the VPN client's aren't kept up-to-date by DHCP. So Edgar and Hank were both in DNS with the same IP. This wouldn't be an issue very often but could be confusing for the client when it does happen.

In thinking of ways to resolve this I decided to increase the interval that DNS deletes stale records. By default this isn't very frequently, about once every 3 days. In true Microsoft fashion there's this [whole complicated process][1] by which the DNS server decides which records are stale enough to delete. I decided it wouldn't be that big of a deal to delete stale records once every 3 hours.

Fast forward about... 3 hours and suddenly our monitoring system is losing connection to several of the client's servers. I check DNS and sure enough about  half the servers' records are gone. After some more digging I found out that by default Windows only re-registers itself with the local DNS server once every 24 hours. This value can be changed using the DefaultRegistrationRefreshInterval registry key (see Microsoft's [KB246804][2]).

Of course I should have checked this before making sweeping changes to the DNS configuration. But that would make me a good administrator instead of just so/so. :)

   [1]: http://technet.microsoft.com/en-us/library/cc759204%28v=ws.10%29.aspx
   [2]: http://support.microsoft.com/kb/246804