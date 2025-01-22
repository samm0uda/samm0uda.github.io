---
id: 634
title: 'Confirm if an invitation is sent to a specific email in Partners Portal / Possibility to resend the invitation'
date: '2021-02-18T08:20:37+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=634'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to resend invitation email to a user added by admins in Partners Portal. This could be used to confirm if an admin has sent an invitation to a specific email address or user, and if so you can resend the email with the invitation link without having the permission to the Partner account.

**Reproduction Steps**

Using the attacker account:

1\) Send a POST request to <https://www.facebook.com/api/graphql/> with valid cookies and those parameters in the body:

__a=1&amp;  
fb_dtsg=CSRF_TOKEN&amp;  
doc_id=2757598104258915&amp;  
variables={“partner_id”:**XXXXXXX**,”portal_type”:”fbs”,”email”:”**YYYYYYY**“}

where **XXXXXXX** is the partner_id of the target partner account, and **YYYYYYY** is the pending email, **fbs** represents the portal type of the Partner account

**Impact**

This could be used to determine admin email addresses added to the Partners portal account and resend a new invitations to those email addresses without having permission or access to the account.

**Timeline**

Feb 5, 2020— Report Sent   
Feb 20, 2020— Acknowledged by Facebook  
Mar 10, 2020— Fixed by Facebook  
Mar 12, 2020 — $500 bounty awarded by Facebook.