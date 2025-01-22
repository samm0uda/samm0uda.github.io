---
id: 38
title: 'Bruteforce Instagram account&#8217;s passwords (lack of rate limiting protection).'
date: '2019-01-22T14:36:57+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=38'
categories:
    - Uncategorized
---

This bug could allowed and attacker to bruteforce any Instagram account password without limiting his attempts. This issue occurs because no Rate Limit Protection is implemented. I was able to make 100 tries per second.

**Proof of Concept**  
A simple POST request to the endpoint **https://www.facebook.com /presence/instagram/connect/** with the parameters **presence_owner_id** with a any page id owned by the attacker and **username** which takes the username of the targeted Instagram account as value and **password** which is the password to try.  
If the provided credentials are correct i should get **payload:null** in the response otherwise the credentials are wrong.

![](http://127.0.0.1:4000/assets/img/2019/01/1_KxA0_qkURq69fodM53Q5gw.png) 

Response if correct credentials were provided  
![](http://127.0.0.1:4000/assets/img/2019/01/1_2qOf_2r17l2ErWxdqIS4OA.png) 

Response if incorrect credentials were provided  
**Fix**  
The Facebook security team fixed this bug by limiting the number of requests to only 20.

**Timeline**  
Mar 1, 2018 — Report Sent  
Mar 3, 2018 — Clarification requested by Facebook  
Mar 3, 2018 — Clarification sent  
Mar 7, 2018 — Further Clarification requested by Facebook  
Mar 7, 2018 — Video sent  
Mar 9, 2018 — Further details sent  
Mar 10, 2018 — Fixed by Facebook