---
id: 308
title: 'Send emails on behalf of legal_noreply@fb.com'
date: '2019-08-01T15:47:05+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=308'
categories:
    - Uncategorized
---

**Description**

This bug could have allowed an attacker to send emails from legal_noreply@fb.com to any email address and change the body of the email . This could be used to send phishing emails to workplace domains admin generally which are familiar with this email to get policy change updates from Facebook .

**Demonstration**

Using an enumeration technique similar to the one discussed in [@phwd write-up](https://philippeharewood.com/facebook-marketing-confidential-call-transcript/), i was able to find an URL used by Facebook employees and their partners to send emails to their Facebook workplace customers/partners from the email legal_noreply@fb.com.  
  
Navigating to this special workflow page **https://legal.tapprd.thefacebook.com/tapprd/Portal/ShowWorkFlow/AnonymousEmbed/XXXXXXXXXXXXX** allowed me to specify a receiving email and a name of a customer where the email template couldn’t be changed.   
However, the customer’s name field wasn’t well secured against HTML injection attacks which allowed me to completely change the body of the email and comment the rest of it.

[![](http://127.0.0.1:4000/assets/img/2019/08/POC_email-1024x335.png)](http://127.0.0.1:4000/assets/img/2019/08/POC_email.png)**Timeline**

Jan 13, 2019— Report Sent  
Jan 16, 2019 — Acknowledged by Facebook  
May 2, 2019 — Fixed and 500$ bounty awarded by Facebook