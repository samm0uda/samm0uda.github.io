---
id: 640
title: 'Expose information about Partner accounts in Partner portal'
date: '2021-02-18T08:45:44+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=640'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to expose some information about a Partner account in Partners Portal by only knowing the partner_id which could be collected online. The information could disclose the partner name, region, locale, country, portal type and admins depending if the partner has subscribers.

**Impact**

This could be used to know if a certain ISP/Mobile and other types of partners, are using Facebook Partner Portal and an attacker may use this information to target these companies by collecting the information from this bug.

**Reproduction Steps**

Send a POST request to <https://www.facebook.com/api/graphql/> without cookies and with the following variable:

q=node(PARTNER_ID){partner_name,partner_unique_name,partner_portal_type,country,locale,region, subscribers,type}

the subscribers may return some ids but depending on portal type. Other fields may be leaking information too but i only got those from bruteforcing the fields.

**Timeline**

May 12, 2020— Report Sent   
Feb 5, 2021— Acknowledged by Facebook:  
 This wasn’t acknowledged at first due to some patches done to GraphQL and the bug was kept unfixed (but can’t be exploited for a while) for less than a year. The Facebook team acknowledged this was valid after reaching them about his again and handled it immediately.  
Feb 5, 2021 — $3600 bounty awarded by Facebook. (including bonuses)  
Feb 11 , 2020— Fixed by Facebook