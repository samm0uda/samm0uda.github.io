---
id: 256
title: 'Disclose the content of internal Facebook Javascript modules.'
date: '2019-04-22T14:14:18+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=256'
categories:
    - Uncategorized
---

This bug could have allowed a malicious user to disclose the content of internal Facebook Javascript modules which have constants/configurations/endpoints of tools used internally by employees. This bug is critical because having access to this data could have let the attacker disclose the internal infrastructure of Facebook which could help him in his next attacks.

**Description**  
  
Accessing Javascript modules of internal tools is normally not possible even if you know the correct name of the module because Facebook checks if the user making the request is authorized to see that tool configurations/constants or not. However, due to lack of verification in one of Facebook subdomains, it was possible to get these modules content ( Internal modules names could be found anywhere if you know where to look )

**Demonstration**  
  
To get the content of a Javascript module we send a GET request to www.facebook.com**/ajax/haste-response/?modules=** or www.facebook.com**/ajax/bootloader-endpoint/?modules=**   
with the correct module name in the “modules” parameter. If you try an internal javascript module you’ll get a “500 Internal Error”.   
To bypass this i tried to send a GET request to the same domain ( www.facebook.com ) but i changed the HOST header in the request to an internal domain like ( REDACTED.facebook.com ) and i kept the same endpoint. As a result, i got the content of the javascript module.  
To do this, i used this simple curl command:  
   
`curl -H 'Host: REDACTED.facebook.com' -H 'Cookie: XX' 'https://www.facebook.com/ajax/bootloader-endpoint/?modules=REDACTED'`

**Timeline**  
  
Feb 6, 2019 — Report Sent  
Feb 8, 2019— Acknowledged by Facebook  
Mar 11, 2019— Fixed by Facebook  
Apr 1, 2019 —Bounty Awarded by Facebook