---
id: 487
title: 'Disclose content of internal Facebook javascript modules ( Revisited )'
date: '2020-07-23T16:56:03+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=487'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

This bug could have allowed a malicious user to disclose the content of internal Facebook Javascript modules which have constants/configurations/endpoints of tools used internally by employees. This bug is critical because having access to this data could have let the attacker disclose the internal infrastructure of Facebook which could help him in his next attacks.

**Description**

Accessing Javascript modules of internal tools is normally not possible even if you know the correct name of the module because Facebook checks if the user making the request is authorized to see that tool configurations/constants or not.

While i was doing some basic enumeration, i encountered the domain **https://internalfb.com** which is used by Facebook emplyees to login ( similar to **https://our.internmc.facebook.com** ). This domain had some /intern/ endpoints accessible to unauthorized users like /intern/login/ and /intern/ajax/bootloader-endpoint/. This bootloader endpoint is similar to /ajax/bootloader-endpoint/ however it would return internal modules too instead of ‘{“error”:true}’.

**Timeline**

May 19, 2020— Report Sent  
May 27, 2020— Acknowledged by Facebook  
Jun 12, 2020— Fixed by Facebook  
July 15, 2020 —Bounty Awarded by Facebook