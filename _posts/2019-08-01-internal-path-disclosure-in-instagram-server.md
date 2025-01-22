---
id: 321
title: 'Internal path disclosure in Instagram server'
date: '2019-08-01T16:27:47+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=321'
categories:
    - Uncategorized
---

**Description**

This bug could have allowed an attacker to raise a PHP exception after manipulating with some parameters in the request which led to the disclosure of internal system paths of the server i.instagram.com and names of some internal methods/classes.

**Demonstration**

1. After enabling users CAs, we launch a proxy which enables us to intercept requests made from the Instagram App.
2. We intercept any POST request to the server **https://i.instagral.com/**.
3. We change the path to **https://i.instagram.com/api/v1/ads/graphql/** and add to the request body this parameter “&amp;doc_id=197633352–“.
4. After this, a PHP exception should be raised which exposes the described information above.

Please notice that this specific endpoint had some serious problems concerning exception handling and many exceptions could been raised to expose other methods/files if we provide a doc_id/query_id of a Query/Mutation that does different action from the other. This happened because some doc_ids returned details about objects that didn’t exist (like the viewer/actor object)”

**Timeline**

Mar 27, 2019— Report Sent  
Mar 28, 2019 — Acknowledged by Facebook  
Jun 18, 2019 — Fixed by Facebook  
Jul 18, 2019 — Bounty Awarded by Facebook