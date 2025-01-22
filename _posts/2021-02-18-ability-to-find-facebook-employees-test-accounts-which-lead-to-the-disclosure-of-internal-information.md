---
id: 638
title: 'Ability to find Facebook employee&#8217;s test accounts which lead to the disclosure of internal information.'
date: '2021-02-18T08:38:26+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=638'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug was not actually a real vulnerability however a weakness in the way employees test accounts user ids were generated. The ids of these accounts are in a certain numerical range which made it easy to extract all the list of these accounts.

**Impact**

The impact here is huge. First, the name of these accounts contains the Controllers or the Gatekeepers the employees are testing. Second, These test accounts usually are friends with the employee main account which could be used to extract a list of FB employees.  
Finally, the content in these accounts like videos/photos exposes very critical information. These information includes employees faces/ employees typing/ devices (phones/modems/computers) serial numbers. Also some of the accounts disclosed tools source code/ testing internal tools / in-development tools, debugging tools, emails, credentials, videos where employees are talking about an idea of a tool and even personal videos of employees or office.

**Reproduction Steps**

Setup  
===  
1\) To get the list of these accounts, i run a fuzzer which make POST request to <https://www.facebook.com/api/graphql/> without cookies to avoid detection and with the parameter q=node(ID){__typename}

The list of ID is the range 499000000 – 499999999.

2\) After getting the list of account ids, i run the fuzzer again to get the list of videos by sending a POST request to <https://www.facebook.com/api/graphql/> with the parameters

variables={“count”:50,”scale”:1,”id”:”BASE_64″}  
&amp;doc_id=2902049703244388

where BASE_64 is a base64 encoded “app_collection:FB_ID:1560653304174514:133” with FB_ID is the test account id. This returned all accounts videos.

3\) Gathering the video_ids from these accounts and then downloading them.

These videos ,as i described above, contained very critical information and below a sample of some of the accounts that disclosed these data:  
<https://www.facebook.com/499645675/videos/10150012296724945/>   
<https://www.facebook.com/bugvideosuser.bugvideosuser/videos>  
<https://www.facebook.com/video.23>  
<https://www.facebook.com/wang189/videos>  
<https://www.facebook.com/ronald.test.52>  
<https://www.facebook.com/499731162/videos/>  
<https://www.facebook.com/jxhhh.jxhhh>

**Timeline**

May 8, 2020— Report Sent   
May 8, 2020— Acknowledged by Facebook  
May 14, 2020 — $500 bounty awarded by Facebook. (seriously?)  
Dec 1 , 2020— Fixed by Facebook