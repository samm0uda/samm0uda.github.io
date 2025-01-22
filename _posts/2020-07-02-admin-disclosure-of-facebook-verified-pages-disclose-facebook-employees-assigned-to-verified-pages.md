---
id: 479
title: 'Admin disclosure of Facebook verified pages/ Disclose Facebook employee assigned to help a verified page.'
date: '2020-07-02T21:42:21+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=479'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to disclose the username of a page admin by only supplying the id of the targeted page. This happens because a certain endpoint would disclose the username of the admin who signed the TOS for having a page sticker (blue tick) or a page that uses Fan subscription ( gaming pages ). Also, the same endpoint leaked the ID and name of the Facebook employee assigned to help the page. This would allowed a malicious user to target a list of verified pages which would disclose a big list of employees Facebook accounts.

**Reproduction**

Send a POST request to https://www.facebook.com/media/manager/fan_sticker_props/ with required parameters like __a and fb_dtsg in the body of the request along with the parameter:  
**page_id**: which would be the id of the targeted page

This request would return the following response:

`"payload":{"pageID":PAGE_ID,"pageName":"PAGE_NAME","spmMemberID":"<strong>EMPLOYEE_ID</strong>","spmMemberName":"<strong>EMPLOYEE_NAME</strong>","tosAcceptTime":"<strong>TIME</strong>","tosAcceptUserName":"<strong>USERNAME</strong>","stickerURL":null,"stickerDescription":null,"stickerFeedback":null,"stickerState":null}`

The interesting fields that were exposing data to unauthorized users in the response were tosAcceptTime, tosAcceptUserName, spmMemberID, and spmMemberName.  
  
The username of the page admin is found in the **tosAcceptUserName** field.  
Information about the Facebook employee assigned to help the verified page like his FBID and his name were exposed in the **spmMemberID** and **spmMemberName**.

**Timeline**

May 30, 2020 — Report Sent for the admin disclosure bug.  
Jun 1, 2020 — Acknowledged by Facebook  
Jun 18, 2020 — Fixed by Facebook  
Jun 25, 2020 — 4500$ Bounty awarded by Facebook  
—  
Jun 25, 2020 — Report Sent for the employee disclosure bug.  
Jun 26, 2020 — Acknowledged by Facebook  
July 1, 2020 — Fixed by Facebook  
July 2, 2020 — 1000$ Bounty awarded by Facebook