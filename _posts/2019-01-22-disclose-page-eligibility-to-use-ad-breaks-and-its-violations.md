---
id: 64
title: 'Disclose page violations and its eligibility to use Ad-breaks'
date: '2019-01-22T21:33:20+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=64'
categories:
    - Uncategorized
---

  
This bug could allowed a malicious user with no role in a page to see if a page is eligible for Facebook Monetization programs like Ad-break. Also he can disclose details about Facebook decision like the number of page violations and details about these violations.

**Reproduction Steps:**  
Send a **POST** request to this endpoint https://www.facebook.com/creator/monetization/eligibility_info/ with the parameter **creator_id** in the request body where creator_id is the id of targeted page.

![](http://127.0.0.1:4000/assets/img/2019/01/P.png)Response of the request sent**Impact**  
This bug is acknowledged by Facebook security team because it could have revealed sensitive information to malicious users like the history of page violations which could harm the page or brand reputation.

**Timeline**  
Sep 11, 2018 — Report Sent  
Sep 12 2018 — Acknowledged by Facebook  
Sep 26, 2018 — Fixed by Facebook  
Oct 2, 2018 — Bounty Awarded by Facebook