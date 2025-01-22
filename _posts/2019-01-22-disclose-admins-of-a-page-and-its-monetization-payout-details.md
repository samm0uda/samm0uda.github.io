---
id: 60
title: 'Disclose page&#8217;s admins and its Monetization payout details'
date: '2019-01-22T21:20:09+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=60'
categories:
    - Uncategorized
---

This bug could have let an unprivileged attacker disclose the admins of a page and the payout account details, so long as the page has Monetization enabled.

**Reproduction steps**  
Send a POST request to this endpoint https://www.facebook.com/media/manager/monetization/payout_settings/  
with the parameter **page_ids** in the request body which takes a list of ids of targeted pages.   
In the response we get the admins list of the page along with payout details like the company name,added date and the financial id.  
The data returned is very sensitive because the financial id could be used in other attacks to expose more details like page incomes.  
Please notice that this attack works only if the page is earning money on Facebook by using Monetization programs like ad-breaks.

**Impact**  
This bug is simple to exploit but it could have revealed very sensitive information to malicious users like admins of a page and its payout details.  
  
**Timeline**  
Sep 11, 2018 — Report Sent  
Sep 11, 2018 — Acknowledged by Facebook  
Sep 14, 2018 — Fixed by Facebook  
Sep 25, 2018 — Bounty Awarded by Facebook