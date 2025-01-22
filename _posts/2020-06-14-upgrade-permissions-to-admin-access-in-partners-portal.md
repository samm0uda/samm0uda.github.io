---
id: 460
title: 'Privilege escalation in Partners Portal to Admin access'
date: '2020-06-14T19:48:59+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=460'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow a malicious user with low permissions in a “Partners Portal” account, to upgrade his access to full admin permissions. This may help him to take full control to delete other admins or have read/write capabilities to do certain actions that may damage the enterprise depending on the portal type (FBS , OPP or Mobile …)

**Reproduction**

1\) Send a POST request to https://www.facebook.com/api/graphql/ with cookies that belongs to a user who has limited access to a Partners Portal account. with those parameters in the body:

`__a=1&<br></br>fb_dtsg=CSRF_TOKEN&<br></br>doc_id=REDACTED&<br></br>variables={"id":"<strong>PARTNER_ID</strong>","partner_type":"<strong>PARTNER_TYPE</strong>"}`

<span class="has-inline-color has-vivid-red-color">PARTNER_ID</span> : targeted partner account id.  
<span class="has-inline-color has-vivid-red-color">PARTNER_TYPE</span> : targeted partner portal account type (eg: fbs, xpp, opp)

This should return a list of permissions groups. The attack get the id associated to the “Admin Access” group which is a group with full permissions. we’ll note this permission group id as **Permission_Group**

2\) Send a POST request to https://www.facebook.com/api/graphql/ with cookies that belongs to a user who has limited access to a Partners Portal account. with those parameters in the body:

`__a=1&<br></br>fb_dtsg=CSRF_TOKEN&<br></br>doc_id=REDACTED&<br></br>variables={"existing_partner_ids":<strong>PARTNER_ID</strong>,"email":"<strong>ATTACKER_EMAIL</strong>","user_id":<strong>ATTACKER_USER_ID</strong>,"group_ids":<strong><strong>PERMISSION_GROUP</strong></strong>}`

<span class="has-inline-color has-vivid-red-color">PARTNER_ID</span> : Targeted partner account id.  
<span class="has-inline-color has-vivid-red-color">ATTACKER_EMAIL</span> : Email address associated to the attacker account in the portal.  
<span class="has-inline-color has-vivid-red-color">ATTACKER_USER_ID</span> : ID of the Facebook account of the attacker  
<span class="has-inline-color has-vivid-red-color">Permission_Group</span> : ID of the “Admin Access” permission group

The response to this request would be (**“update_user_permissions”: true**). This request would work too to change the permissions to other admin accounts even if our account doesn’t have permissions to update them.

**Impact**

A low-privileged Partner Portal account user can escalate their privileges to the highest/admin level.

**Timeline**

Feb 5, 2020 — Report Sent  
Feb 7, 2020 — Acknowledged by Facebook  
Feb 24, 2020 — Fixed by Facebook  
Feb 26, 2020 — Bounty awarded by Facebook