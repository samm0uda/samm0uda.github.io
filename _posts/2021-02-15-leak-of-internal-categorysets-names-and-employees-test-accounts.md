---
id: 613
title: 'Leak of internal categorySets names and employees test accounts.'
date: '2021-02-15T08:51:18+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=613'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

An attacker could read some internal data which meant to be private and seen by Facebook employees only. The data accessed by the attacker consists of current Facebook Translation categories and projects and each project available locales and also information about tags and employees test accounts (points of contact).

**Impact**

Categories and tags names may reveals upcoming Facebook Projects because the data returned are translation projects/tasks for each Facebook project. Also the data returned contains ids for employees test accounts and tags names and ids. The information leaked should be visible to Facebook employees only.

**Reproduction Steps**

Request 1:  
Send a POST request to this endpoint : [https://www.facebook.com/translations/tasks/request/shared_info/](https://www.facebook.com/translations/tasks/request/shared_info/) with the required cookies and parameters in the body of the request (__a=1&amp;&amp;fb_dtsg=) and as response we get all information described above.

Request 2:  
Same data could be accessed by sending a POST request to this endpoint [https://www.facebook.com/translations/tags_typeahead/](https://www.facebook.com/translations/tags_typeahead/) with the parameter ( value=**xxx** ) in the request body where **xxx** in the project/tag name we want to search for. ( __a=1&amp;&amp;fb_dtsg=\*\*\* must be present in the body )

**Timeline**

Dec 15, 2018— Report Sent   
Dec 17, 2018— Acknowledged by Facebook  
Sep 19, 2019— Fixed by Facebook  
Sep 25, 2019 — $500 bounty awarded by Facebook.