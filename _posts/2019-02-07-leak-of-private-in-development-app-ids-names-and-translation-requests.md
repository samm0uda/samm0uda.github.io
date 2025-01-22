---
id: 171
title: 'Leak of private/in-development app ids, names and translation requests'
date: '2019-02-07T20:54:30+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=171'
categories:
    - Uncategorized
---

This bug could allowed a malicious user to extract a large list of Facebook in-development apps along with their names, ids and translation requests.  
This bug is critical by my opinion because it could be used by malicious parties to steal ideas of these apps before their publication, this is done based on translation requests for their names and description. Also, the ids could possibly used to exploit other bugs.  
  
**Demonstration**  
I found this bug by accessing a deprecated dashboard designed to manage translation requests for apps developers. This dashboard could be accessed in

```
GET /translations/admin/?app_id=XXXXXXXXXX <br></br>Host: www.facebook.com
```

  
after specifying an owned app id (XXXXXXXXXX).  
Normally this panel would return only translation requests of the specified app in the URL but it returned all apps with their name, ids and translation requests even if these apps are still in development mode (Nearly 20000 apps were leaked).  
What i also noticed that even if the app developer didn't made a translation request, the app still appears because he has added the "App Center" product in the app dashboard.   
  
**Timeline**  
Dec 28, 2018 — Report Sent  
Jan 2, 2019—Clarification requested by Facebook  
Jan 2, 2019 — Clarification sent  
Jan 7, 2019—Clarification requested by Facebook  
Jan 7, 2019 — Clarification sent  
Jan 9, 2019 — Acknowledged by Facebook  
Jan 18, 2019 — Fixed by Facebook  
Jan 25, 2019 — Bounty Awarded by Facebook  