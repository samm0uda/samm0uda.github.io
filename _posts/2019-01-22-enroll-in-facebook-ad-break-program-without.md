---
id: 68
title: 'Enroll in Facebook Ad-break program without Facebook approval'
date: '2019-01-22T22:01:39+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=68'
categories:
    - Uncategorized
---

The Ad-break program is designed for certain users to use it to add ads in videos posted in their pages. To enroll in this program, the page should be eligible and comply with Monetization eligibility standards (Number of followers , specific countries and audience …).  
However this bug could have allowed a user to enroll in this program without complying with those standards and get paid for ads in videos that aren’t qualified or viewed by certain people in certain countries (the audience in a specific country doesn’t engage with the ads like other countries).   
So this could have caused a huge money lose to Facebook if it was exploited in mass scale by malicious users.

**Reproduction Steps:**  
Send a POST request to this endpoint https://www.facebook.com/mm/onboarding/postdata/ with those parameters in the body of the request:

- selected_page_ids : OWNED_PAGE_ID
- business_id : OWNED_BUSINESS_ID
- financial_entity_id : OWNED_FINANCIAL_ID

After a successful attack, If we browse to the page settings we should see a Monetization tab.  
Now If we post a 3-min video we should also see an ad-break feature available for us. (We could manage this feature from Creator Studio also).

**Impact**  
This allowed an attacker to circumvent Facebook’s eligibility checks and enroll their page in Ad Breaks.

**Timeline**  
Sep 21, 2018 — Report Sent  
Sep 24, 2018—Clarification requested by Facebook  
Sep 25, 2018 — Clarification sent  
Sep 27, 2018 — Acknowledged by Facebook  
Oct 4, 2018 — Fixed by Facebook  
Oct 24, 2018 — Bounty Awarded by Facebook