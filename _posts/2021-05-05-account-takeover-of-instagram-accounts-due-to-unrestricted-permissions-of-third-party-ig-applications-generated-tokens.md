---
id: 684
title: 'Account takeover of Instagram accounts due to unrestricted permissions of third-party application&#8217;s generated tokens'
date: '2021-05-05T14:35:18+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=684'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

Access tokens returned when an Instagram user authorize a third-party Instagram application which was created to use the Instagram Basic Display API, could be used to access **graph.instagram.com/graphql** endpoint which allows the owner of the application to make Query/Mutations and bypass the API permissions given or in some scenarios full takeover the account.

**Reproduction Steps**

According to the [documentation](https://developers.facebook.com/docs/instagram-basic-display-api/getting-started) of the Instagram Basic Display API, any user could create an Instagram application that would request from the user permissions like user, media . After the creation, the application should be approved by Facebook. If the application review was successful, the application owner could then redirect Instagram users from his website to Instagram authorization page and if the user chose to use the application, he should receive an Instagram access token

<figure class="wp-block-image size-full is-style-default">![](http://127.0.0.1:4000/assets/img/2021/05/73104138_383070672579531_6594576748594069504_n.png)The access token generated and returned to the website owner is supposed to only have access to a limited set of APIs endpoints to query some user information or media as [documented by Facebook](https://developers.facebook.com/docs/instagram-basic-display-api/guides/getting-profiles-and-media) . However, it seems that due to some recent changes to the API, the access tokens generated where able to access more Graph APIs like **graph.instagram.com/graphql** endpoint and also could be used in some Facebook.com queries/mutations that directly lead to the possibility to takeover the Instagram account or Page/Business accounts linked to it.

Finally, to explain how this could be abused :  
– An attacker could create a normal application and start to embed the Instagram Application authorization request in his website. Since the authorization page only shows that the application would only have access to limited permissions, the victim would accept the usage and though the attacker can takeover his account.  
– Since the API allows Long-Lived Access Tokens, a website owner that uses Instagram for authentication or offer a service for Instagram insights and analytics, would have already a big Instagram user base that he could target using this to gain information or takeover their accounts. Of course, this won’t require victim interaction since they already authorized the application

**Proof of concept**

1. Victim gets redirected to **https://www.instagram.com/oauth/authorize?app_id=ATTACKER_APP_ID&amp;response_type=code&amp;redirect_uri=https://attacker.website/&amp;scope=user_profile**
2. Victim authorize the third-party Instagram application and accept to give basic permissions like user_profile
3. Attacker receives the access_token and for example try to make this GraphQL Mutation which would disconnect the Instagram account from Facebook pages ( This example was given because the Account takeover approach is more complicated and also still could be helpful to me in future bugs 🙂 )  
     **https://graph.instagram.com/graphql?access_token=IG_THIRD_PARTY_TOKEN&amp;doc_id=3131323080272171&amp;method=POST**

**Impact**

Instagram third-party application access tokens could be used to perform unauthorized mutations that could lead to Instagram account takeover or Facebook business and page takeover.   
This bug was fixed and no evidence of abuse was found according to Facebook.

**Timeline**

Mar 15, 2021— Report Sent   
Mar 18, 2021— Acknowledged by Facebook  
Apr 14, 2021— Fixed by Facebook  
May 4, 2021 — $18K bounty awarded by Facebook ( including bonus )