---
id: 691
title: 'Identify a Facebook user by his phone number despite privacy settings set'
date: '2021-05-06T23:15:18+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=691'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to identify if a phone number is linked to a Facebook user account and if so what’s the id of the user. While adding a phone number in **m.facebook.com** to the attacker Facebook account, the endpoint **m.facebook.com/phoneacquire/** would return the current owner of the phone number despite the privacy settings set by the owner.

**Reproduction Steps**

1\) From the attacker account, go to **https://m.facebook.com/ntdelegatescreen/?params={“saved”:true}&amp;path=/contacts/management/**  
2\) Add a new new phone number that you need to look up if it’s linked to a Facebook account  
3\) A redirect to **https://m.facebook.com/phoneacqwrite/** endpoint should be done. In the attached parameters, there’s a parameter called **giver_id** which would be the user id of the Facebook user who has this phone number added to his account.

**Impact**

This could have been misused to deanonymize/identify a Facebook user account linked to given phone number.

**Timeline**

Mar 13, 2021— Report Sent   
Mar 17, 2021— Acknowledged by Facebook  
Apr 7, 2021— Fixed by Facebook  
Apr 26, 2021 — $9K bounty awarded by Facebook (Including bonus)