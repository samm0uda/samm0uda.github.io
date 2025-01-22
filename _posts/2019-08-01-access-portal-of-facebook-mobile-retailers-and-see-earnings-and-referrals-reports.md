---
id: 314
title: 'Access portal of Facebook mobile retailers and see earnings and referrals reports.'
date: '2019-08-01T16:12:31+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=314'
categories:
    - Uncategorized
---

**Description**

This bug could have allowed a malicious user to access dashboard designed for certain mobile carriers to refer to Facebook platforms. This dashboard contains a list of referred users and their activity on Facebook. This occurred because no authentication to access the dashboard was present and the attacker has to only know the retailer_id which was easy to guess.   
Also the attacker could have used the retailer account to register new customers by providing their numbers then after they sign up using the invitation link, he can spy on them by seeing their last actions like if they added friends, created pages or added photos.

**Demonstration**

- The vulnerable endpoint is : **https://m.facebook.com/f123/report/?retailer_id**.   
    To access the account information, a user has to only get a valid retailer_id which could be bruteforced based on a one exposed or leaked online, the problem here is that no authentication process or a parameter with special hash is present.   
    After enumeration, i was able to retrieve a valid retailer_id {REDECATED} and using this one i was able to retrieve nearly 1000 others by a bruteforcing technique i found.  
    This dashboard exposed last registrations and actions for those accounts.

[![](http://127.0.0.1:4000/assets/img/2019/08/Acc11-1024x529.png)](http://127.0.0.1:4000/assets/img/2019/08/Acc11.png)- In this page we can see that a phone number is present next to “Facebook Expert Mobile Number:”. Using this number we can get this retailer earning and refers to other customers and then spy on their activities.  
    This could be done by navigating to:  
     **https://m.facebook.com/expert/?retailer_mobile=XXXX**   
    where XXXX is the retrieved number. We can then choose to view financial report or to add new customers.

[![](http://127.0.0.1:4000/assets/img/2019/08/Acc22-1024x197.png)](http://127.0.0.1:4000/assets/img/2019/08/Acc22.png)Retailer login by entering their number.[![](http://127.0.0.1:4000/assets/img/2019/08/Acc33-1024x254.png)](http://127.0.0.1:4000/assets/img/2019/08/Acc33.png)Retailer earnings and referralsFacebook fixed this bug by allowing access to the retailer account only if the entered phone matches your current account phone number.

**Timeline**

Feb 6, 2019— Report Sent  
Feb 7, 2019 — Acknowledged by Facebook  
Jun 20, 2019 — Fixed by Facebook  
Jul 11, 2019 — Bounty Awarded by Facebook