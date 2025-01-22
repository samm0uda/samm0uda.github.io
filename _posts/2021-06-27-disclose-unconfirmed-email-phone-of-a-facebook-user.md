---
id: 700
title: 'Disclose unconfirmed email/phone of a Facebook user'
date: '2021-06-27T09:33:09+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=700'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could have allowed an attacker to target Facebook users in order to potentially leak unconfirmed emails or phone numbers. A link should be visited by the targeted user in order for this to work.  This happens because the endpoints **https://www.facebook.com/n/** and **https://www.facebook.com/nd/** would make certain redirects based on parameters supplied. This endpoint is mainly used in emails sent from Facebook for email/mobile confirmations or updates. If the combination of parameters “**mid**” and “**code**” are supplied, the endpoint would redirect to the path selected by the attacker (explained below) and append to it, unconfirmed or newly confirmed email (or phone number)

**Details**

1. The example link is as follow: https://www.facebook.com/n/**?/billing_interfaces/external_result/&amp;mid=XXXXXXXX&amp;code=YYYYYYYY**
2. The first parameter name would be the next path to redirect to and the one that should have the contactpoint appended to it.
3. **XXXXXXXX** is a string that would contain a hex encoded user id of the victim. It has this format:  
    AAAAAAAAAAAAA**G**5b011f85facd**G**1111111111111**G**AAA. Notice the second string if we use **G** as a separator “5b011f85facd”. This should be replaced by a hex encoded user id. Here the random user id “100060381969101” was encoded which should be the target user id.  
      
    Notice that the **mid** parameter must contain the hex encoded user id which should match the Facebook logged-in user id (target). This could be achieved and exploited in two ways:  
    – We would firstly extract the user id of a target and then hex encoded before sending him/her the link.  
    – In order to this to work for any Facebook user that visits the attacker website, this could chained with another bug that would leak the Facebook user id of a website visitor.  
      
    **YYYYYYYY** could be set randomly.
4. If the previous condition is met ( encoded userid in mid matches the logged-in user id ), a redirect should be made to **https://www.facebook.com**/billing_interfaces/external_result/**** with the parameter **contactpoint** containing any unconfirmed email/phone number.   
    The reason i chose ****/billing_interfaces/external_result/**** endpoint because it would return a script that would send the current URL with parameters to opener/parent window via postMessage. We should setup an eventListener for cross-window messages to catch it.  
      
    **Proof of concept**

```markup
<html>
<head>
<script>window.addEventListener("message",function a(event){alert(event.data.direct_debit_redirect_url.split("=")[2]);});</script>
</head>
<body>
<iframe width="1" height="1" src="<strong>https://www.facebook.com/n/?/billing_interfaces/external_result/&mid=AAAAAAAAAAAAAG5b011f85facdG1111111111111GAAA&code=01</strong>"/>
</body>
</html>
```

**Impact**

This could be used to leak user unconfirmed email/phone number. As you may know, many Facebook users usually add an email which is never created and use phone number confirmation instead. This bug could open an attack surface for malicious users to know which email is added in the Facebook account and then create it. The second stage would be to confirm it by adding another script in the website visited by the victim. This would allow account takeovers of Facebook accounts.

**Timeline**

Dec 20, 2020— Report Sent   
Dec 29, 2020— Acknowledged by Facebook  
Feb 16, 2021 — $500 bounty awarded by Facebook (?).  
Feb 23, 2021— Fixed by Facebook

</body></html>