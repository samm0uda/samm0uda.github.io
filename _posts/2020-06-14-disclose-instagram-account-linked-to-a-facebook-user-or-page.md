---
id: 455
title: 'Disclose the Instagram account linked to a Facebook user account or page'
date: '2020-06-14T19:20:40+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=455'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to disclose the Instagram account id linked to a Facebook user or page and the name of this account.

**Reproduction**

1\) Send a POST request to https://www.facebook.com/api/graphql/ with required CSRF parameters and those additional parameters in the request body:

doc_id=2505810016176167  
&amp;variables={“ownerId”:**X**}

where **X** is the id of the targeted page or user. This would return in the response the linked account id (shadow_id), which could be enumerated using Facebook Graph API, and the name the Instagram account.

**Impact**

The shadow_id of the Instagram account disclosed of a the Facebook user or page could be potentially used in other bugs like IDORs where the id of the targeted account is needed. Or simply the impact here is disclosing the Instagram account linked to a user or page which is normally private and only known ( in certain cases ) by the user or the page admins.

**Timeline**

May 3, 2020 — Report Sent  
May 8, 2020 — Acknowledged by Facebook  
May 12, 2020 — Fixed by Facebook  
May 14, 2020 — Bounty awarded by Facebook