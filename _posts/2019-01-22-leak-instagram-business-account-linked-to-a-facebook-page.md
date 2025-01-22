---
id: 56
title: 'Disclose Instagram business account linked to a Facebook page'
date: '2019-01-22T21:02:47+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=56'
categories:
    - Uncategorized
---

A malicious user with no permissions in a page can see its associated Instagram business account. The leaked data is meant to be visible to page admins only and when exposed it can help the attacker in further attacks.   
  
**Reproduction Steps**  
To reproduce this, we send a **POST** request to the endpoint https://www.facebook.com/business/instagram/lightweight_business_info/ with the parameter **page_id** in the body of the request where page_id is the ID of the targeted page.  
In the response, we get the page Instagram Business account information if of course there is one linked to the page.

**Impact** The information exposed are sensitive and private because it could help the attacker to harm the page business or access more private data if they’re used in other attacks.

**Timeline**  
Aug 31, 2018 — Report Sent  
Sep 4, 2018 — Clarification requested by Facebook  
Sep 4, 2018 — Clarification sent  
Sep 4, 2018 — Further Clarification requested by Facebook  
Sep 5, 2018 — Video sent  
Sep 7, 2018 — Acknowledged by Facebook  
Sep 17, 2018 — Fixed by Facebook  
Oct 17, 2018 — Bounty Awarded by Facebook